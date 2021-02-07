1.链码开发
先设计一个简单的应用场景，假设有这样的业务需求：

可以添加学校，信息包括学校名称、学校ID；
添加该学校的学生，信息包括姓名，用户ID，所在学校ID，所在班级名称；
更新学生信息；
根据学生ID查询学生信息；
根据学生ID删除学生信息；
根据学校ID删除学校信息，包括该学校下的所有学生。
接下来开发满足这些需求的链码。关键代码如下 
1.1 定义School，Student结构体

type StudentChaincode struct {
}

type Student struct {
    UserId      int    `json:"user_id"` //学生id
    Name        string `json:"name"` //姓名
    SchoolId    string `json:"school_id"` //学校id
    Class       string `jsong:"class"` //班级名称
}

type School struct {
    SchoolId    string `json:"id"` //学校id
    School      string `json:"name"` //学校名称
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
1.2 部分业务需求的实现 
1.2.1 添加学校 
这里用到了shim.ChaincodeStubInterface的CreateCompositeKey，来创建联合主键。其实Fabric就是用U+0000来把各个联合主键的字段拼接起来，因为这个字符太特殊，所以很适合，对应的拆分联合主键的字段的方法是SplitCompositeKey。

func (t *StudentChaincode) initSchool(stub shim.ChaincodeStubInterface, args []string) pd.Response {
    if len(args) != 2 {
        return shim.Error("Incorrect number of arguments. Expecting 2(school_id, school_name)")
    }

    schoolId := args[0]
    schoolName := args[1] 
    school := &School{schoolId, schoolName}

    //这里利用联合主键，使得查询school时，可以通过主键的“school”前缀找到所有school
    schoolKey, err := stub.CreateCompositeKey("School", []string{"school", schoolId})
    if err != nil {
        return shim.Error(err.Error())
    }

    //结构体转json字符串
    schoolJSONasBytes, err := json.Marshal(school)
    if err != nil {
        return shim.Error(err.Error())
    }
    //保存
    err = stub.PutState(schoolKey, schoolJSONasBytes)
    if err != nil {
        return shim.Error(err.Error())
    }
    return shim.Success(schoolJSONasBytes)
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
1.2.2 添加学生，需要检查所属学校是否已经存在。 
抽出来的工具类方法querySchoolIds 中用到的GetStateByPartialCompositeKey 是对联合主键进行前缀匹配的查询

func (t *StudentChaincode) addStudent(stub shim.ChaincodeStubInterface, args []string) pd.Response {
    st, err := studentByArgs(args)
    if err != nil {
        return shim.Error(err.Error())
    }

    useridAsString := strconv.Itoa(st.UserId)

    //检查学校是否存在，不存在则添加失败
    schools := querySchoolIds(stub)
    if len(schools) > 0 {
        for _, schoolId := range schools {
            if schoolId == st.SchoolId {
                goto SchoolExists;
            }
        }
        fmt.Println("school " + st.SchoolId+ " does not exist")
        return shim.Error("school " + st.SchoolId+ " does not exist")
    } else {
        fmt.Println("school " + st.SchoolId+ " does not exist")
        return shim.Error("school " + st.SchoolId+ " does not exist")
    }

    SchoolExists:
    //检查学生是否存在
    studentAsBytes, err := stub.GetState(useridAsString)
    if err != nil {
        return shim.Error(err.Error())
    } else if studentAsBytes != nil {
        fmt.Println("This student already exists: " + useridAsString)
        return shim.Error("This student already exists: " + useridAsString)
    }

    //结构体转json字符串
    studentJSONasBytes, err := json.Marshal(st)
    if err != nil {
        return shim.Error(err.Error())
    }
    //保存
    err = stub.PutState(useridAsString, studentJSONasBytes)
    if err != nil {
        return shim.Error(err.Error())
    }

    return shim.Success(studentJSONasBytes)
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
//将参数构造成学生结构体
func studentByArgs(args []string) (*Student, error) {
    if len(args) != 4 {
        return nil, errors.New("Incorrect number of arguments. Expecting 4(name, userid, schoolid, classid)")
    }

    name := args[0]
    userId, err := strconv.Atoi(args[1]) //字符串转换int
    if err != nil {
        return nil, errors.New("2rd argument must be a numeric string")
    }
    schoolId := args[2]
    class := args[3]
    st := &Student{userId, name, schoolId, class}

    return st, nil
}

//获取所有创建的学校id
func querySchoolIds(stub shim.ChaincodeStubInterface) []string {
    resultsIterator, err := stub.GetStateByPartialCompositeKey("School", []string{"school"})
    if err != nil {
        return nil
    }
    defer resultsIterator.Close()

    scIds := make([]string,0)
    for i := 0; resultsIterator.HasNext(); i++ {
        responseRange, err := resultsIterator.Next()
        if err != nil {
            return nil
        }
        _, compositeKeyParts, err := stub.SplitCompositeKey(responseRange.Key)
        if err != nil {
            return nil
        }
        returnedSchoolId := compositeKeyParts[1]
        scIds = append(scIds, returnedSchoolId)
    }
    return scIds
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
1.2.3 删除学校，包括删除所有对应学生信息 
删除学校下的所有学生时，需要根据根据学校ID找到匹配的所有学生。这里用到了富查询方法GetQueryResult ，可以查询value中匹配的项。但如果是LevelDB，那么是不支持，只有CouchDB时才能用这个方法。

func (t *StudentChaincode) deleteSchool(stub shim.ChaincodeStubInterface, args []string) pd.Response {
    if len(args) < 1 {
        return shim.Error("Incorrect number of arguments. Expecting 1(schoolid)")
    }
    schoolidAsString := args[0]

    schoolKey, err := stub.CreateCompositeKey("School", []string{"school", schoolidAsString})
    if err != nil {
        return shim.Error(err.Error())
    }

    schoolAsBytes, err := stub.GetState(schoolKey)
    if err != nil {
        return shim.Error("Failed to get school:" + err.Error())
    } else if schoolAsBytes == nil {
        return shim.Error("School does not exist")
    }
    //删除学校
    err = stub.DelState(schoolKey) 
    if err != nil {
        return shim.Error("Failed to delete school:" + schoolidAsString + err.Error())
    }
    //删除学校下的所有学生
    queryString := fmt.Sprintf("{\"selector\":{\"school_id\":\"%s\"}}", schoolidAsString)
    resultsIterator, err := stub.GetQueryResult(queryString)//富查询，必须是CouchDB才行
    if err != nil {
        return shim.Error("Rich query failed")
    }
    defer resultsIterator.Close()
    for i := 0; resultsIterator.HasNext(); i++ {
        responseRange, err := resultsIterator.Next()
        if err != nil {
            return shim.Error(err.Error())
        }
        err = stub.DelState(responseRange.Key)
        if err != nil {
            return shim.Error("Failed to delete student:" + responseRange.Key + err.Error())
        }
    }
    return shim.Success(nil)
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
1.2.4 删除学生

func (t *StudentChaincode) deleteStudent(stub shim.ChaincodeStubInterface, args []string) pd.Response {
    if len(args) < 1 {
        return shim.Error("Incorrect number of arguments. Expecting 1(userid)")
    }
    useridAsString := args[0]
    studentAsBytes, err := stub.GetState(useridAsString)
    if err != nil {
        return shim.Error("Failed to get student:" + err.Error())
    } else if studentAsBytes == nil {
        return shim.Error("Student does not exist")
    }

    err = stub.DelState(useridAsString)
    if err != nil {
        return shim.Error("Failed to delete student:" + useridAsString + err.Error())
    }
    return shim.Success(nil)
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
1.2.5 更新学生信息 
对于State DB来说，增加和修改数据是统一的操作，因为State DB是一个Key Value数据库，如果我们指定的Key在数据库中已经存在，那么就是修改操作，如果Key不存在，那么就是插入操作。

func (t *StudentChaincode) updateStudent(stub shim.ChaincodeStubInterface, args []string) pd.Response {
    st, err := studentByArgs(args)
    if err != nil {
        return shim.Error(err.Error())
    }
    useridAsString := strconv.Itoa(st.UserId)

    //检查学校是否存在，不存在则添加失败
    schools := querySchoolIds(stub)
    if len(schools) > 0 {
        for _, schoolId := range schools {
            if schoolId == st.SchoolId {
                goto SchoolExists;
            }
        }
        fmt.Println("school " + st.SchoolId+ " does not exist")
        return shim.Error("school " + st.SchoolId+ " does not exist")
    } else {
        fmt.Println("school " + st.SchoolId+ " does not exist")
        return shim.Error("school " + st.SchoolId+ " does not exist")
    }

    SchoolExists:
    //因为State DB是一个Key Value数据库，如果我们指定的Key在数据库中已经存在，那么就是修改操作，如果Key不存在，那么就是插入操作。
    studentJSONasBytes, err := json.Marshal(st)
    if err != nil {
        return shim.Error(err.Error())
    }
    //保存
    err = stub.PutState(useridAsString, studentJSONasBytes)
    if err != nil {
        return shim.Error(err.Error())
    }

    return shim.Success(studentJSONasBytes)
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
1.2.6 链码的Init、Invoke实现

func (t *StudentChaincode) Init(stub shim.ChaincodeStubInterface) pd.Response {
    return shim.Success(nil)
}

func (t *StudentChaincode) Invoke(stub shim.ChaincodeStubInterface) pd.Response {

    fn, args := stub.GetFunctionAndParameters()
    fmt.Println("invoke is running " + fn)

    if fn == "initSchool" {
        return t.initSchool(stub, args)
    } else if fn == "addStudent" {
        return t.addStudent(stub, args)
    } else if fn == "queryStudentByID" {
        return t.queryStudentByID(stub, args)
    } else if fn == "deleteSchool" {
        return t.deleteSchool(stub, args)
    } else if fn == "updateStudent" {
        return t.updateStudent(stub, args)
    }

    fmt.Println("invoke did not find func: " + fn) 
    return shim.Error("Received unknown function invocation")

}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
完整代码：https://github.com/mh4u/chaincode_demo

2.单元测试
在开发完链码后，我们并不需要在区块链环境中部署链码才能进行调试。可以利用shim.MockStub 来编写单元测试代码，直接在无网络的环境中debug。

先新建一个编写测试代码的go文件，如student_test.go。假如我们想测试前面的创建学习功能，可以这样码代码：

func TestInitSchool(t *testing.T) {
    //模拟链码部署
    scc := new(StudentChaincode)
    stub := shim.NewMockStub("StudentChaincode", scc)
    mockInit(t, stub, nil)
    //调用链码
    initSchool(t, stub, []string{"schoolId_A", "学校1"})
    initSchool(t, stub, []string{"schoolId_B", "学校2"})
}

func mockInit(t *testing.T, stub *shim.MockStub, args [][]byte) {
    res := stub.MockInit("1", args)
    if res.Status != shim.OK {
        fmt.Println("Init failed", string(res.Message))
        t.FailNow()
    }
}

func initSchool(t *testing.T, stub *shim.MockStub, args []string) {
    res := stub.MockInvoke("1", [][]byte{[]byte("initSchool"), []byte(args[0]), []byte(args[1])})
    if res.Status != shim.OK {
        fmt.Println("InitSchool failed:", args[0], string(res.Message))
        t.FailNow()
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
完整代码：https://github.com/mh4u/chaincode_demo

3. 开发环境测试链码
经过单元测试后，我们还需要在区块链开发环境中跑链码进行测试。 
3.1 下载fabric-samples 
3.2 下载或拷贝二进制文件到fabric-samples目录下 
这里写图片描述
3.3 将开发好的链码文件拷贝到fabric-samples/chaincode目录下 
3.4 进入链码开发环境目录

cd fabric-samples/chaincode-docker-devmode
1
3.5 打开3个终端，每个都进入到chaincode-docker-devmode文件夹 
终端 1 – 启动网络

$ docker-compose -f docker-compose-simple.yaml up
1
终端 2 – 编译和部署链码

$ docker exec -it chaincode bash 
$ cd Student
$ go build
$ CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 ./Student
1
2
3
4
终端3 – 使用链码 
启动cli

$ docker exec -it cli bash
1
安装、实例化链码

$ cd ../
$ peer chaincode install -p chaincodedev/chaincode/Student -n mycc -v 0
$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":[]}' -C myc
1
2
3
调用链码

$ peer chaincode invoke -n mycc -c '{"Args":["initSchool", "schoolId_A", "学校A"]}' -C myc
$ peer chaincode invoke -n mycc -c '{"Args":["addStudent", "张三", "1", "schoolId_A", "classA"]}' -C myc