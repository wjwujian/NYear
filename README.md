# NYear.ODA
## 简介
C# .Net Database ORM for Oracle DB2 MySql SqlServer SQLite MariaDB;<br/>
NYear.ODA 是一个数据库访问的 ORM 组件，能够通用常用的数据库 DB2、Oracle、SqlServer、MySql(MariaDB)、SQLite;<br/>
对不常用的数据库Informix、Sybase、Access也能简单的使用；<br/>
就目前而言，NYear.ODA 是支持 SQL 语法最完整的 C# ORM 组件;对于分库分表、或分布式数据库也留有很好的扩展空间。 <br/>
分页、动态添加条件、子查询、无限连接查询、Union、Group by、having、In子查询、Exists、Insert子查询、Import高速导入都不在话下，<br/>
递归查询、case when、Decode、NullDefault、虚拟字段、数据库function、update 字段运算、表达式等都是 ODA 很有特色的地方。<br/>
允许用户注入SQL代码段，允许用户自己编写SQL代码等，同时也支持存储过程Procedure(或oracle的包）<br/>
由于很多开发者都比较喜欢 Lambda 的直观简单，ODA 的查询也扩展了此功能。<br/>

ODA 的标准功能已经足够强大了，用之开发一套完整 MES 系统或 WorkFlow【工作流】系统都不需要自定义SQL;<br/>
**注： 已有实际项目应用，且Oracle、Mysql、SqlServer 三个数据库上随意切换，**<br/>
**当然，建表时需要避免各种数据库的关键字及要注意不同数据库对数据记录的大小写敏感问题。**<br/>

## NYear.ODA 语法
ODA使用的是链式编程语法，编写方式SQL语法神似；<br/>
开发时如有迷茫之处，基本可以用SQL语句类推出ODA的对应写法。<br/>
ODA为求通用各种数据库，转换出来的SQL都是标准通用的SQL语句；一些常用但数据不兼容的部分，在ODA内部实现（如递归树查询、分页等)。 <br/>
NYear.ODA以 Select、Insert、Update、Delete、Procedure 方法为最终执行方法，调用这些方法时，ODA 将会把生成的SQL语句发送给数据库运行。

## 用法示例
###  查询
#### 简单查询
在简单查询的语句里 Where 方法与 And 方法是等效的，往后很多时候都一样。
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
object data = U.Where(U.ColUserAccount == "User1")
       .And(U.ColIsLocked == "N")
       .And(U.ColStatus == "O")
       .And(U.ColEmailAddr.IsNotNull)  
       .Select(U.ColUserAccount, U.ColUserPassword.As("PWD"), U.ColUserName, U.ColPhoneNo, U.ColEmailAddr); 
```
#### 查询默认实体

```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
List<SYS_USER> data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColStatus == "O", U.ColEmailAddr.IsNotNull)
               .SelectM(U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr);
```
#### 查询并返回指定实体类型
返回的实体类型可以是任意自定义类型，并不一定是对应数据库的实体
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
List<SYS_USER> data = U.Where(U.ColUserAccount == "User1")
               .And(U.ColIsLocked == "N")
               .And(U.ColStatus == "O")
               .And(U.ColEmailAddr.IsNotNull)
               .Select<SYS_USER>(U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr);
```
#### 查询分页
```C#
ODAContext ctx = new ODAContext(); 
int total = 0; 
var U = ctx.GetCmd<CmdSysUser>();
var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
    .SelectM(0,20,out total, U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr); 
```
#### 查询第一行
```C#
ODAContext ctx = new ODAContext(); 
var U = ctx.GetCmd<CmdSysUser>();
var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
    .SelectDynamicFirst(U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr);
            
    string UserName = data.USER_NAME;///属性 USER_NAME 与 ColUserName 的ColumnName一致，如果没有数据则返回null
 ```
#### 返回动态数据模型
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
    .SelectDynamic(U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr);

string UserName = "";
if (data.Count > 0)
UserName =  data[0].USER_NAME; ///与 ColUserName  的 ColumnName一致.
```
#### 去重复 Distinct
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var data = U.Where( U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
    .Distinct.Select(U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr);
```
#### 连接查询
支持 InnerJoin、LeftJoin、RightJion ；可以无限Join
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var R = ctx.GetCmd<CmdSysRole>();
var UR = ctx.GetCmd<CmdSysUserRole>();

var data = U.InnerJoin(UR, U.ColUserAccount == UR.ColUserAccount, UR.ColStatus == "O")
    .InnerJoin(R, UR.ColRoleCode == R.ColRoleCode, R.ColStatus == "O")
    .Where(U.ColStatus == "O",R.ColRoleCode == "Administrator")
    .Select<UserDefineModel>(U.ColUserAccount.As("UserAccount"), U.ColUserName.As("UserName"),R.ColRoleCode.As("Role"), R.ColRoleName.As("RoleName"));
```
#### 简单内连接 
内连接有很多人只使用 join , 但对形如 SELECT t1.* FROM TABLE1 T1,TABLE2 T2,TABLE3 T3,TABLE4 这种写法比较陌生，<br/>
但个人觉得这种连接的写法比较简单，而且容易阅读。
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var R = ctx.GetCmd<CmdSysRole>();
var UR = ctx.GetCmd<CmdSysUserRole>();
var data =  U.ListCmd(UR,R)
    .Where(U.ColUserAccount == UR.ColUserAccount, 
           UR.ColStatus == "O",
           UR.ColRoleCode == R.ColRoleCode,
           R.ColStatus == "O",
           U.ColStatus == "O",
           R.ColRoleCode == "Administrator")
.Select< UserDefineModel>(U.ColUserAccount.As("UserAccount"), U.ColUserName.As("UserName"),U.ColEmailAddr.As("Email"), R.ColRoleCode.As("Role"), R.ColRoleName.As("RoleName"));
/*
SELECT T0.USER_ACCOUNT AS UserAccount,
       T0.USER_NAME    AS UserName,
       T0.EMAIL_ADDR   AS Email,
       T1.ROLE_CODE    AS Role,
       T1.ROLE_NAME    AS RoleName
  FROM SYS_USER T0, SYS_USER_ROLE T2, SYS_ROLE T1
 WHERE T0.USER_ACCOUNT = T2.USER_ACCOUNT
   AND T2.STATUS = @T3
   AND T2.ROLE_CODE = T1.ROLE_CODE
   AND T1.STATUS = @T4
   AND T0.STATUS = @T5
   AND T1.ROLE_CODE = @T6;
*/

```
#### 嵌套子查询
嵌套子查询需要把一个查询子句转换成视图(ToView方法)，转换成视图之后可以把它视作普通的Cmd使用。<br/>
视图里ViewColumns是视图字段的集合。
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var R = ctx.GetCmd<CmdSysRole>();
var UR = ctx.GetCmd<CmdSysUserRole>();

var UA = ctx.GetCmd<CmdSysUserAuthorization>();
var RA = ctx.GetCmd<CmdSysRoleAuthorization>();

var Admin = U.InnerJoin(UR, U.ColUserAccount == UR.ColUserAccount, UR.ColStatus == "O")
    .InnerJoin(R, UR.ColRoleCode == R.ColRoleCode, R.ColStatus == "O")
    .Where(U.ColStatus == "O")
    .ToView(U.ColUserAccount.As("SYS_USER"), U.ColUserName, R.ColRoleCode.As("SYS_ROLE"), R.ColRoleName); ////子查询

var data =  Admin.InnerJoin(UA, UA.ColUserAccount == Admin.ViewColumns[1],UA.ColIsForbidden == "N")
    .InnerJoin(RA,RA.ColRoleCode == Admin.ViewColumns[2],RA.ColIsForbidden =="N") 
    .Where(Admin.ViewColumns[1] == "张三",
           Admin.ViewColumns[2] == "Administrator"
     ).Select(); 
/*
SELECT *
  FROM (SELECT T0.USER_ACCOUNT AS SYS_USER,
               T0.USER_NAME,
               T1.ROLE_CODE    AS SYS_ROLE,
               T1.ROLE_NAME
          FROM SYS_USER T0
         INNER JOIN SYS_USER_ROLE T2
            ON T0.USER_ACCOUNT = T2.USER_ACCOUNT
           AND T2.STATUS = 'O'
         INNER JOIN SYS_ROLE T1
            ON T2.ROLE_CODE = T1.ROLE_CODE
           AND T1.STATUS = 'O'
         WHERE T0.STATUS = 'O') T6
 INNER JOIN SYS_USER_AUTHORIZATION T3
    ON T3.USER_ACCOUNT = T6.USER_NAME
   AND T3.IS_FORBIDDEN =  'N'
 INNER JOIN SYS_ROLE_AUTHORIZATION T4
    ON T4.ROLE_CODE = T6.SYS_ROLE
   AND T4.IS_FORBIDDEN = 'N'
 WHERE T6.USER_NAME = '张三'
   AND T6.SYS_ROLE = 'Administrator';
   */
```
#### Union UnionAll
Union 语句要求被Union或UnionAll的是视图。要求视图与查询的字段的数据库类型及顺序及数据一致（数据库本身的要求，非ODA要求)。
```C#
ODAContext ctx = new ODAContext(); 
var U = ctx.GetCmd<CmdSysUser>(); 
var UR = ctx.GetCmd<CmdSysUserRole>();  
var RA = ctx.GetCmd<CmdSysRoleAuthorization>();
var RS = ctx.GetCmd<CmdSysResource>();

var U1 = ctx.GetCmd<CmdSysUser>();
var UA = ctx.GetCmd<CmdSysUserAuthorization>();
var RS1 = ctx.GetCmd<CmdSysResource>();

U.InnerJoin(UR, U.ColUserAccount == UR.ColUserAccount, UR.ColStatus == "O")
 .InnerJoin(RA, RA.ColRoleCode == UR.ColRoleCode, RA.ColStatus == "O")
 .InnerJoin(RS, RS.ColId == RA.ColResourceId, RS.ColStatus == "O")
 .Where(U.ColUserAccount == "User1");

U1.InnerJoin(UA, U1.ColUserAccount == UA.ColUserAccount, UA.ColStatus == "O")
  .InnerJoin(RS1, RS1.ColId == UA.ColResourceId , RS1.ColStatus=="O")
  .Where(U1.ColUserAccount == "User1");
 
var data = U.Union(U1.ToView(U1.ColUserAccount, U1.ColUserName, UA.ColIsForbidden,
                RS1.ColId, RS1.ColResourceType, RS1.ColResourceScope, RS1.ColResourceLocation
      )).Select(U.ColUserAccount, U.ColUserName, RA.ColIsForbidden,
                RS.ColId, RS.ColResourceType, RS.ColResourceScope, RS.ColResourceLocation
                ); 
/*
SELECT T0.USER_ACCOUNT,
       T0.USER_NAME,
       T2.IS_FORBIDDEN,
       T3.ID,
       T3.RESOURCE_TYPE,
       T3.RESOURCE_SCOPE,
       T3.RESOURCE_LOCATION
  FROM SYS_USER T0
 INNER JOIN SYS_USER_ROLE T1
    ON T0.USER_ACCOUNT = T1.USER_ACCOUNT
   AND T1.STATUS = 'O'
 INNER JOIN SYS_ROLE_AUTHORIZATION T2
    ON T2.ROLE_CODE = T1.ROLE_CODE
   AND T2.STATUS = 'O'
 INNER JOIN SYS_RESOURCE T3
    ON T3.ID = T2.RESOURCE_ID
   AND T3.STATUS = 'O'
 WHERE T0.USER_ACCOUNT = 'User1'
UNION
SELECT T4.USER_ACCOUNT,
       T4.USER_NAME,
       T5.IS_FORBIDDEN,
       T6.ID,
       T6.RESOURCE_TYPE,
       T6.RESOURCE_SCOPE,
       T6.RESOURCE_LOCATION
  FROM SYS_USER T4
 INNER JOIN SYS_USER_AUTHORIZATION T5
    ON T4.USER_ACCOUNT = T5.USER_ACCOUNT
   AND T5.STATUS = 'O'
 INNER JOIN SYS_RESOURCE T6
    ON T6.ID = T5.RESOURCE_ID
   AND T6.STATUS = 'O'
 WHERE T4.USER_ACCOUNT = 'User1';
 */         
```
#### 查询排序
OrderbyAsc 或OrderbyDesc 对数据按顺序或倒序排列，先给出的排序条件优先排。
OrderbyAsc 或OrderbyDesc 参数可以是多个字段。
```C#
ODAContext ctx = new ODAContext();
var RS = ctx.GetCmd<CmdSysResource>();
var datra = RS.Where(RS.ColResourceType == "WEB", RS.ColStatus == "O")
    .OrderbyDesc(RS.ColResourceIndex)
    .SelectM();
```
#### 数据库查询条件语法
Join、Where、Having 查询参数：条件之间可用运算符 “|”(Or方法）或“&”(And方法)表明条件与条件之间的关系；“|”和“&”只能用于字段的运算 <br/>
如同时列出多个条件，则说明SQL语句要同时满足所有条件（即“And”关系）；<br/>
Where和Having方法可以多次调用，每调一次SQL语句累加一个条件（And、Or、Groupby、OrderbyAsc、OrderbyDesc方法类同)；<br/>
与 Where 方法同等级的 And 方法与 Where 方法是等效的;数据筛选条件可以根据业务情况动态增加；<br/>
IS NULL/ IS NOT NULL 条件可由字段直接带出，如：ColEmailAddr.IsNotNull <br/>
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var UR = ctx.GetCmd<CmdSysUserRole>();

var data = U.InnerJoin(UR, U.ColUserAccount == UR.ColUserAccount, UR.ColStatus == "O")
    .Where(U.ColStatus == "O", U.ColEmailAddr.IsNotNull.Or(U.ColEmailAddr == "riwfnsse@163.com"), U.ColIsLocked == "N")
    .Where(UR.ColRoleCode.In("Administrator", "Admin", "PowerUser", "User", "Guest")) 
    .Groupby(UR.ColRoleCode)
    .Having(U.ColUserAccount.Count > 2)
    .OrderbyAsc(U.ColUserAccount.Count)
    .Select(U.ColUserAccount.Count.As("USER_COUNT"), UR.ColRoleCode);

//以下写法是等效的，条件可以不断的累加
U.InnerJoin(UR, U.ColUserAccount == UR.ColUserAccount, UR.ColStatus == "O");
U.Where(U.ColStatus == "O");
U.Where(U.ColEmailAddr.IsNotNull.Or(U.ColEmailAddr == "riwfnsse@163.com"));
U.And(U.ColIsLocked == "N");
U.Where(UR.ColRoleCode.In("Administrator", "Admin", "PowerUser", "User", "Guest"));
U.Groupby(UR.ColRoleCode);
U.Having(U.ColUserAccount.Count > 2);
U.OrderbyAsc(U.ColUserAccount.Count);
data = U.Select(U.ColUserAccount.Count.As("USER_COUNT"), UR.ColRoleCode);
```
#### 分组统计, Groupby  Having
Groupby 、Having、OrderbyAsc 方法里支持 Function 运算；
```C#
 ODAContext ctx = new ODAContext();
 var U = ctx.GetCmd<CmdSysUser>();
 var UR = ctx.GetCmd<CmdSysUserRole>();
 var data = U.InnerJoin(UR, U.ColUserAccount == UR.ColUserAccount, UR.ColStatus == "O")
     .Where(U.ColStatus == "O", UR.ColRoleCode.In("Administrator", "Admin", "PowerUser", "User", "Guest"))
     .Groupby(UR.ColRoleCode)
     .Having(U.ColUserAccount.Count > 2)
     .OrderbyAsc(UR.ColRoleCode,U.ColUserAccount.Count)
     .Select(U.ColUserAccount.Count.As("USER_COUNT"), UR.ColRoleCode);
```

#### IN/NOT IN 条件
IN/NOT IN 有两个重载，一个是in数组，一个是in子查询
```C#
ODAContext ctx = new ODAContext(); 
var RA = ctx.GetCmd<CmdSysRoleAuthorization>();
var RS = ctx.GetCmd<CmdSysResource>(); 
///IN 数组
RA.Where(RA.ColIsForbidden == "N", RA.ColStatus == "O", RA.ColRoleCode.In("Administrator", "Admin", "PowerUser"));

///IN 子查询
var data = RS.Where(RS.ColStatus == "O", RS.ColId.In(RA, RA.ColResourceId)) 
    .SelectM(); 
```
#### Exists/NOT Exists 子查询
```C#
ODAContext ctx = new ODAContext();
var RA = ctx.GetCmd<CmdSysRoleAuthorization>(); 
//Exists 子查询的条件
var RS = ctx.GetCmd<CmdSysResource>();
RA.Where(RA.ColIsForbidden == "N", RA.ColStatus == "O", RA.ColResourceId == RS.ColId); 

var data = RS.Where(RS.ColStatus == "O", RS.Function.Exists(RA, RA.AllColumn)) 
    .SelectM();
```
#### 递归查询
ODA 递归查询效果与Oracle的StartWith ConnectBy语句一致。<br/>
ODA 处理原理：先以 where 条作查出需要递归筛先的数据，然后在内存中递归筛选。<br/>
由于是在内存递归，所以递归使所用到的所有字段必须包含在 Seclect 字段里。<br/>
注：ODA 递归性能比 oracle 数据库的 StartWith ConnectBy 差一个等级，但比 SQLServer 的 with as 好一个级等。<br/>
递归有深度限制，ODA 限制最大深度是31层。<br/>
被递归的原始数据越多性能下降很快，最好保被递归筛选的数在10W条以内 
```C#
ODAContext ctx = new ODAContext();
////由根向叶子递归 Prior 参数就是递归方向
CmdSysResource RS = ctx.GetCmd<CmdSysResource>();  
var rlt = RS.Where(RS.ColStatus == "O", RS.ColResourceType == "MENU")
    .StartWithConnectBy(RS.ColResourceName.ColumnName + "='根菜单'", RS.ColParentId.ColumnName, RS.ColId.ColumnName, "MENU_PATH", "->", 10)
.Select(RS.ColResourceName.As("MENU_PATH"), RS.ColId, RS.ColParentId, RS.ColResourceName, RS.ColResourceType, RS.ColResourceScope, RS.ColResourceLocation, RS.ColResourceIndex);
           
////由叶子向根递归,Prior 参数就是递归方向
CmdSysResource RS1 = ctx.GetCmd<CmdSysResource>(); 
var rlt1 = RS.Where(RS.ColStatus == "O", RS.ColResourceType == "MENU")
    .StartWithConnectBy(RS.ColResourceName.ColumnName + "='菜单1'", RS.ColId.ColumnName, RS.ColParentId.ColumnName, "MENU_PATH", "<-", 10)
.Select(RS.ColResourceName.As("MENU_PATH"), RS.ColId, RS.ColParentId, RS.ColResourceName, RS.ColResourceType, RS.ColResourceScope, RS.ColResourceLocation, RS.ColResourceIndex);
```
#### Lambda语法支持
Lambda 语法是由 ODA 原生语法扩展而来的，ODA 使用者也可以自行扩展。<br/>
ODA 原生语法是可以无限连接的，但目前 Lambda 语法支持最多九个表的连接查询。
```C#
int total = 0;
var data = new ODAContext().GetJoinCmd<CmdSysUser>()
   .InnerJoin<CmdSysUserRole>((u, ur) => u.ColUserAccount == ur.ColUserAccount & ur.ColStatus == "O")
   .InnerJoin<CmdSysRole>((u, ur, r) => ur.ColRoleCode == r.ColRoleCode & r.ColStatus == "O")
   .InnerJoin<CmdSysRoleAuthorization>((u, ur, r, ra) => r.ColRoleCode == ra.ColRoleCode & ra.ColIsForbidden == "O" & ra.ColStatus == "O")
   .Where((u, ur, r, ra) => u.ColStatus == "O" & (r.ColRoleCode == "Administrator" | r.ColRoleCode == "Admin") & u.ColIsLocked == "N")
   .Groupby((u, ur, r, ra) => new IODAColumns[] { r.ColRoleCode, u.ColUserAccount })
   .Having((u, ur, r, ra) => ra.ColResourceId.Count > 10)
   .OrderbyAsc((u, ur, r, ra) => new IODAColumns[] { ra.ColResourceId.Count })
   .Select(0, 20, out total, (u, ur, r, ra) => new IODAColumns[] { r.ColRoleCode, u.ColUserAccount, ra.ColResourceId.Count.As("ResourceCount") });        
```

### 更新数据
#### 通常的 update 方式
Update 的 where 条件与 查询是语句是一致的。
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColStatus == "O", U.ColEmailAddr.IsNotNull)
 .Update(U.ColUserName == "新的名字", U.ColIsLocked == "Y");
```
#### 模型数据 Upadte
使用实体 Update 数据时，对于属性值为 null 的字段不作更新。<br/>
这是由于在 ORM 组件在实际应用中，多数时候界面回传的是完整的实体对象，<br/>
或者收接时使用完整的实体作为反序列化的容器，那些不需要更新的字段也在其中，而且为null。<br/>
```C#
ODAContext ctx = new ODAContext(); 
var U = ctx.GetCmd<CmdSysUser>();
    U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColStatus == "O", U.ColEmailAddr.IsNotNull)
    .Update(new SYS_USER()
    {
        ADDRESS = "自由国度",
        CREATED_BY = "InsertModel",
        CREATED_DATE = DateTime.Now, 
        STATUS = "O",
        USER_ACCOUNT = "NYear1",
        USER_NAME = "多年1",
        USER_PASSWORD = "123",
        IS_LOCKED = "N",
    });
```
#### 更新运算
 支持的运算符号：+ 、 - 、*、/、%
 目前对一个字段更新时，只支持一个运算符号；
```C#
ODAContext ctx = new ODAContext(); 
var U = ctx.GetCmd<CmdSysUser>();
var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
    .Update(U.ColFailTimes == U.ColFailTimes + 1, U.ColUserName == U.ColUserAccount + U.ColEmailAddr ); 
```
#### 删除数据
Delete的where条件  SELECT 语句一致
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
   .Delete();
```
### 插入数据据
#### 插入指定字段的数据
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
U.Insert(U.ColStatus == "O", U.ColCreatedBy == "User1", U.ColLastUpdatedBy == "User1", U.ColLastUpdatedDate == DateTime.Now,
        U.ColCreatedDate == DateTime.Now,U.ColUserAccount == "Nyear", U.ColUserName == "多年", U.ColUserPassword == "123",
        U.ColFeMale == "M", U.ColFailTimes ==0,U.ColIsLocked =="N");
```
#### 插入模型的数据
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>(); 
U.Insert(new SYS_USER()
  {
      ADDRESS = "自由国度",
      CREATED_BY = "InsertModel",
      CREATED_DATE = DateTime.Now, 
      FAIL_TIMES = 0,
      STATUS = "O",
      USER_ACCOUNT = "NYear1",
      USER_NAME = "多年1",
      USER_PASSWORD = "123",
      IS_LOCKED ="N",
  }); 
```
#### 批量导入数据d 
导入 DataTable 数据时，要保证 DataTable 字段类型与数据库对应的字段类型一致
```C#
DataTable data = new DataTable();
data.Columns.Add(new DataColumn("ADDRESS"));
data.Columns.Add(new DataColumn("CREATED_BY"));
data.Columns.Add(new DataColumn("CREATED_DATE",typeof(DateTime)));
data.Columns.Add(new DataColumn("EMAIL_ADDR"));
data.Columns.Add(new DataColumn("LAST_UPDATED_BY"));
data.Columns.Add(new DataColumn("LAST_UPDATED_DATE", typeof(DateTime))); 
data.Columns.Add(new DataColumn("FAIL_TIMES", typeof(decimal)));
data.Columns.Add(new DataColumn("STATUS"));
data.Columns.Add(new DataColumn("DUMMY"));
data.Columns.Add(new DataColumn("USER_ACCOUNT"));
data.Columns.Add(new DataColumn("USER_NAME"));
data.Columns.Add(new DataColumn("USER_PASSWORD"));
data.Columns.Add(new DataColumn("IS_LOCKED"));
             
for (int i = 0; i < 10000; i++)
{
    object[] dr = new object[]
    {
        "自由国度",
         "User1" ,
         DateTime.Now,
         "riwfnsse@163.com",
         "User1" ,
         DateTime.Now,
         0,
         "O",
         "Dummy",
         "ImportUser" + i.ToString(),
         "导入的用户" + i.ToString(),
         "123",
         "N"                   
      };
      data.Rows.Add(dr); 
}

 ODAContext ctx = new ODAContext();
 var U = ctx.GetCmd<CmdSysUser>();
 U.Import(data);
```
### 函数
#### 数据库函数
ODA提供数据库常用的通用系统函数：MAX, MIN,  COUNT, SUM, AVG, LENGTH, LTRIM, RTRIM, TRIM, ASCII, UPPER,  LOWER <br/>
这些函数由字段直接带出，如：ColUserAccount.Count <br/>
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
object data = U.Where(U.ColStatus == "O", U.ColIsLocked == "N")
       .Groupby(U.ColUserAccount)
       .Select(U.Function.Count.As("CountAll"), U.ColUserAccount.Count.As("CountOne"), U.ColUserAccount.Upper.As("UPPER_ACC"), U.ColUserAccount.Trim.Ltrim.As("TRIM_ACC"));

/*
SELECT COUNT(*) AS CountAll,COUNT(T0.USER_ACCOUNT) AS CountOne,UPPER(T0.USER_ACCOUNT) AS UPPER_ACC,LTRIM(T0.USER_ACCOUNT) AS TRIM_ACC FROM SYS_USER T0 WHERE T0.STATUS = 'O' AND T0.IS_LOCKED = 'N' GROUP BY T0.USER_ACCOUNT;
 */
       
```
#### 表达式
Express方法, 用户可在 SELECT 字段中注入自定义的一段SQL脚本。
因为ODA 的表达式是开发者注入的一段SQL语句，所以SQL注入的风险及是否可以跨数据库，就用开发者掌握了。
```C#
ODAParameter p1 = new ODAParameter() { ColumnName = "Params1", DBDataType = ODAdbType.OVarchar, Direction = System.Data.ParameterDirection.Input, ParamsName = ODAParameter.ODAParamsMark + "Params1", ParamsValue = "我是第一个参数的值", Size = 2000 };
 ODAParameter p2 = new ODAParameter() { ColumnName = "Params2", DBDataType = ODAdbType.OVarchar, Direction = System.Data.ParameterDirection.Input, ParamsName = ODAParameter.ODAParamsMark + "Params2", ParamsValue = "这是SQL语句注入", Size = 2000 };
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
object data = U.Where(U.ColStatus == "O", U.ColIsLocked == "N")
       .Select(U.Function.Express("1+1").As("COMPUTED"),
        U.Function.Express(" null ").As("NULL_COLUMN"), 
        U.Function.Express(" 'Function( + " + ODAParameter.ODAParamsMark + "Params1, " + ODAParameter.ODAParamsMark + "Params2)' ", p1, p2).As("SQL_Injection"));
 
 /*
SELECT 1+1 AS COMPUTED, null  AS NULL_COLUMN, 'Function( + @Params1, @Params2)'  AS SQL_Injection 
FROM SYS_USER T0 WHERE T0.STATUS = @T1 AND T0.IS_LOCKED = @T2;
 */
    
```
#### 虚拟字段、临时字段
VisualColumn 方法是对 Express 方法的再次封装，为应用提供方便，免出数据转换麻烦、避免SQL注入风险、保证数据库通用。
```C#
 ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
object data = U.Where(U.ColStatus == "O", U.ColIsLocked == "N")
      .Select(U.Function.VisualColumn("HELLO , I am NYear software").As("STRING_COLUMN"), U.Function.VisualColumn(DateTime.Now).As("APPLICATION_DATETIME"), U.Function.VisualColumn(0).As("DIGIT_COLUMN"));
 /*
 SELECT 'HELLO , I am NYear software' AS STRING_COLUMN,@T1 AS APPLICATION_DATETIME,0 AS DIGIT_COLUMN 
 FROM SYS_USER T0 
 WHERE T0.STATUS = 'O' AND T0.IS_LOCKED = 'N'; 
   */
```
#### 用户自定义的函数
CreateFunc 方法,用可在 SELECT 字段中加入自定义的数据库函数，但不同的数据库对调用自定义函数的方法差异太大，ODA无法将其统一。<br/>
所以若要ODA Function.CreateFunc方法也要以跨数据库，则需要在创建数据库时，特殊处理数据库的schema,user,dbowner,database等对象的名称。 <br/>
```C#
ODAContext ctx = new ODAContext();
var RS = ctx.GetCmd<CmdSysResource>();
object data = RS.Where(RS.ColStatus == "O",RS.ColResourceType =="MENU") 
       .Select(RS.AllColumn,RS.Function.CreateFunc("dbo.GET_RESOURCE_PATH", RS.ColId).As("RESOURCE_PATH"));
 /*
 SELECT T0.*,dbo.GET_RESOURCE_PATH(T0.ID) AS RESOURCE_PATH 
 FROM SYS_RESOURCE T0 
 WHERE T0.STATUS = 'O' AND T0.RESOURCE_TYPE = 'MENU';
 */
```
#### 数据转内容转换 CaseWhen
SQL 语句： case when  条件 then  值 when 条件 then 值 else 默认值 end 
```C#
ODAContext ctx = new ODAContext();  
var U = ctx.GetCmd<CmdSysUser>();
Dictionary<ODAColumns, object> Addr = new Dictionary<ODAColumns, object>();
Addr.Add(U.ColAddress.IsNull, "无用户地址数据...");
Addr.Add(U.ColAddress.Like("%公安局%"), "被抓了?");

Dictionary<ODAColumns, object> phone = new Dictionary<ODAColumns, object>();
phone.Add(U.ColPhoneNo.IsNull, "这个家伙很懒什么都没有留下");
phone.Add(U.ColPhoneNo == "110", "小贼快跑");
phone.Add(U.ColAddress.NotLike("%公安局%"), "被抓了?");

object data = U.Where(U.ColStatus == "O", U.ColIsLocked == "N")
       .Select(U.Function.CaseWhen(Addr, U.ColAddress).As("ADDRESS"), U.Function.CaseWhen(phone, "110").As("PHONE_NO"));
       
 /*
 SELECT (CASE
         WHEN T0.ADDRESS IS NULL THEN
          '无用户地址数据...'
         WHEN T0.ADDRESS LIKE '%公安局%' THEN
          '被抓了?'
         ELSE
          T0.ADDRESS
       END) AS ADDRESS,
       (CASE
         WHEN T0.PHONE_NO IS NULL THEN
          '这个家伙很懒什么都没有留下'
         WHEN T0.PHONE_NO = '110' THEN
          '小贼快跑'
         WHEN T0.ADDRESS NOT LIKE '%公安局%' THEN
          '被抓了'
         ELSE
         '110'
       END) AS PHONE_NO
  FROM SYS_USER T0
 WHERE T0.STATUS = 'O'
   AND T0.IS_LOCKED ='N';
*/
       
```
#### 空值转换
NullDefault 是对CaseWhen方法的再次封装，以方便应用
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
object data = U.Where(U.ColStatus == "O", U.ColIsLocked == "N")
       .Select(U.Function.NullDefault(U.ColAddress, "无用户地址数据...").As("ADDRESS"), U.Function.NullDefault(U.ColPhoneNo,110).As("PHONE_NO"));
       
/*
SELECT ( CASE  WHEN T0.ADDRESS IS NULL  THEN '无用户地址数据...' ELSE T0.ADDRESS END ) AS ADDRESS,
( CASE  WHEN T0.PHONE_NO IS NULL  THEN 110  ELSE T0.PHONE_NO END ) AS PHONE_NO 
FROM SYS_USER T0 WHERE T0.STATUS = 'O' AND T0.IS_LOCKED = 'N';
*/
```
#### 数据转内容转换 Case
SQL 语句： case 字段 when  对比值 then 值 when 对比值 then 值 else 默认值 end 
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();

Dictionary<object, object> Addr = new Dictionary<object, object>();
Addr.Add(U.Function.Express(" NULL "), "无用户地址数据...");
Addr.Add("天堂", "人生最终的去处");
Dictionary<object, object> phone = new Dictionary<object, object>();
phone.Add(U.Function.Express(" NULL "), "这个家伙很懒什么都没有留下");
phone.Add( "110", "小贼快跑");
phone.Add(U.ColAddress, "资料有误，电话与地址相同");

object data = U.Where(U.ColStatus == "O", U.ColIsLocked == "N")
       .Select(U.Function.Case(U.ColAddress,Addr, U.ColAddress).As("ADDRESS"), U.Function.Case(U.ColPhoneNo,phone, U.ColPhoneNo).As("PHONE_NO"));
/*
SELECT (CASE T0.ADDRESS
         WHEN NULL THEN
          '无用户地址数据...'
         WHEN '天堂' THEN
          '人生最终的去处'
         ELSE
          T0.ADDRESS
       END) AS ADDRESS,
       (CASE T0.PHONE_NO
         WHEN NULL THEN
          '这个家伙很懒什么都没有留下'
         WHEN '110' THEN
          '小贼快跑'
         WHEN T0.ADDRESS THEN
          '资料有误，电话与地址相同'
         ELSE
          T0.PHONE_NO
       END) AS PHONE_NO
  FROM SYS_USER T0
 WHERE T0.STATUS = 'O'
   AND T0.IS_LOCKED = 'N'

*/
 
```
#### 数据转内容转换Decode
ODA Decode方法 模拟Oracle内置Decode函数,对Case方法的再次封装，以方便应用
```C#
ODAContext ctx = new ODAContext();
var RS = ctx.GetCmd<CmdSysResource>();
object data = RS.Where(RS.ColStatus == "O", RS.ColResourceType == "MENU")
       .Select(RS.Function.Decode(RS.ColResourceType, "未知类型", "WEB", "网页资源", "WFP_PAGE", "WPF页面资源", "WPF_WIN", "WPF程序窗口", "WIN_FORM", "FORM窗口").As("RESOURCE_TYPE")
                 , RS.AllColumn); 
                 
/*
SELECT ( CASE T0.RESOURCE_TYPE WHEN 'WEB' THEN '网页资源' WHEN 'WFP_PAGE' THEN 'WPF页面资源' 
WHEN 'WPF_WIN' THEN 'WPF程序窗口' WHEN 'WIN_FORM' THEN 'FORM窗口' ELSE '未知类型' END )  AS RESOURCE_TYPE,T0.* FROM SYS_RESOURCE T0 WHERE T0.STATUS ='O' AND T0.RESOURCE_TYPE = 'MENU';
*/
                 
```

### 进阶应用
这里介绍一些在开发过程中不时会遇到的问题，对应有处理方式。   

#### 字段的连接
  字段的连接，不同数据库的处理差异太大，ODA没有提供字符串连接的方法<br/>
  但可以用户DataTable方法或通能过实体属性实现<br/>
  ```C#
 DataTable dt = new DataTable();
 dt.Columns.Add(new DataColumn("COL_ID", typeof(string)));
 dt.Columns.Add(new DataColumn("COL_NUM", typeof(int)));
 dt.Columns.Add(new DataColumn("COL_TEST", typeof(string)));
 dt.Columns.Add(new DataColumn("COL_NUM2", typeof(int)));
 for (int i = 0; i < 100; i++) 
 {
     if(i%3==1)
      dt.Rows.Add(Guid.NewGuid().ToString("N").ToUpper(), i + 1, string.Format("this is {0} Rows", i + 1), null);
      else
      dt.Rows.Add(Guid.NewGuid().ToString("N").ToUpper(), i + 1, string.Format("this is {0} Rows", i + 1), 1000);
} 
dt.Columns.Add("CONNECT_COL", typeof(string), "COL_ID+'  +  '+COL_TEST");
dt.Columns.Add("ADD_COL", typeof(decimal), "COL_NUM+COL_NUM2");
```
#### Datatable转List
```C#
 DataTable data = new DataTable();
 data.Columns.Add(new DataColumn("ADDRESS"));
 data.Columns.Add(new DataColumn("CREATED_BY"));
 data.Columns.Add(new DataColumn("CREATED_DATE", typeof(DateTime)));
 data.Columns.Add(new DataColumn("EMAIL_ADDR"));
 data.Columns.Add(new DataColumn("LAST_UPDATED_BY"));
 data.Columns.Add(new DataColumn("LAST_UPDATED_DATE", typeof(DateTime)));
 data.Columns.Add(new DataColumn("FAIL_TIMES", typeof(decimal)));
 data.Columns.Add(new DataColumn("STATUS"));
 data.Columns.Add(new DataColumn("DUMMY"));
 data.Columns.Add(new DataColumn("USER_ACCOUNT"));
 data.Columns.Add(new DataColumn("USER_NAME"));
 data.Columns.Add(new DataColumn("USER_PASSWORD"));
 data.Columns.Add(new DataColumn("IS_LOCKED"));
 for (int i = 0; i < 10000; i++)
 {
     object[] dr = new object[]
     {
        "自由国度",
        "User1" ,
        DateTime.Now,
        "riwfnsse@163.com",
        "User1" ,
        DateTime.Now,
        0,
        "O",
        "Dummy",
        "ImportUser" + i.ToString(),
        "导入的用户" + i.ToString(),
        "123",
        "N"
      };
    data.Rows.Add(dr);
}
List<SYS_USER> DataList = ODA.DBAccess.ConvertToList<SYS_USER>(data); 
```
#### 自定义SQL
如果SQL语可以重复使用，或者为有程序更规范，推荐派生 ODACmd 类 重写SQL生成方法。<br/>
ODA 提供一个通用的派生类 SQLCmd ，可执行临时的SQL语句。<br/>
```C#
ODAContext ctx = new ODAContext();
var sql = ctx.GetCmd<SQLCmd>();
var data = sql.Select("SELECT * FROM SYS_USER WHERE USER_ACCOUNT = @T1", ODAParameter.CreateParam("@T1","User1"));
////可用方法
/// sql.Select<T>(string Sql, params ODAParameter[] Parameters)
/// sql.SelectDynamicFirst(string Sql, params ODAParameter[] Parameters)
/// sql.Update(string Sql, params ODAParameter[] Parameters)
/// sql.Insert(string Sql, params ODAParameter[] Parameters)
/// sql.Delete(string Sql, params ODAParameter[] Parameters)
/// sql.Procedure(string Sql, params ODAParameter[] Parameters)
  
```
#### 自定义存储过程
如果SQL语可以重复使用，或者为有程序更规范，推荐派生 ODACmd 类 重写SQL生成方法
```C#
ODAContext ctx = new ODAContext();
var sql = ctx.GetCmd<SQLCmd>();
var data = sql.Procedure("");
```  
#### SQL Debug 调试
开发者要查看 ODA 最终执行的 SQL 及其参数值，可以在调试时查看 ODAContext.LastODASQL 静态属性。<br/>
```C#
ODAContext ctx = new ODAContext();
var U = ctx.GetCmd<CmdSysUser>();
var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
    .SelectDynamicFirst(U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr); 
////ODAContext.LastODASQL;属性是ODA最近生成的SQL语句块,包含了ODA 控制SQL语句(并非真正发送给数据库的SQL)、参数、语句类型、操作对象等； 
var ODASQL = ODAContext.LastODASQL;
///ctx.LastSQL属性是最近发送给数据库的SQL；由于分页的方法是两条SQL，所以此处的SQL是最后读取数据库的SQL；
string sql = ctx.LastSQL;
///ctx.SQLParams属性是最近发送给数据库的SQL的参数；
object[] param = ctx.SQLParams;  
```
#### ODA钩子
如果有数据库有分库分表，ODA 提供的由算法不合适，可通过 ODA 钩子自定义数据库、表的路由<br/>
当然用户可以通 ODA 的钩子程序可以监控到所有 ODA 执行的 SQL及其参数，这为业务程序问题定位及日志记录供了一个方便的入口<br/>
钩子程序里，无需进行SQL语句分析,就有明确的 SQL 类型，需要访问的表及字段信息供使用。程序这对数据库的操作一清二楚。
```C#
public static object Hook()
{
  ///开发者可以通过ODA钩子自定义SQL路由,在SQL执行前对SQL进行修改； 
  ODAContext.CurrentExecutingODASql += ODASqlExecutingEvent; 
  DAContext ctx = new ODAContext();
  var U = ctx.GetCmd<CmdSysUser>();
  var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
     .Select(U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr); 
     ODAContext.CurrentExecutingODASql -= ODASqlExecutingEvent;
     return data;
}

private static void ODASqlExecutingEvent(object source, ExecuteEventArgs args)
{
    if (args.SqlParams.ScriptType == SQLType.Select
      && args.SqlParams.TableList.Contains("SYS_USER"))
    {
       args.DBA = new ODA.Adapter.DbASQLite("Data Source=./sqlite.db"); ///改变ODA预设的数据库执行实例，重新实例化一个SQL语句执行实例。
       args.SqlParams.ParamList.Clear();
       args.SqlParams.SqlScript.Clear();
       args.SqlParams.SqlScript.AppendLine(" SELECT * FROM SYS_ROLE"); ///修改将要执行的SQL语句
    }
} 
  
///SQL语句监控钩子
public static object Monitor()
{
   ///开发者可能通过此钩子，可以监控所有发送给数据库SQL语句及其参数。 
   ODAContext.CurrentExecutingSql += SqlExecutingEvent;
   ODAContext ctx = new ODAContext();
   int total = 0;
   var U = ctx.GetCmd<CmdSysUser>();
   var data = U.Where(U.ColUserAccount == "User1", U.ColIsLocked == "N", U.ColEmailAddr.IsNotNull)
      .Select(0, 20, out total, U.ColUserAccount, U.ColUserName, U.ColPhoneNo, U.ColEmailAddr);
    ODAContext.CurrentExecutingSql -= SqlExecutingEvent;
    return data;
}
private static void SqlExecutingEvent(string Sql, object[] prms)
{
    ///记录将要被执行的SQL语句及其参数
    string LogSql = Sql + prms.ToString();  
}
```

### ODA开发工具应用
#### ODA 提供三个工具：<br/>
第一个当然是 ODA 的代码生成工具了，它生成 ODA 需要的实体类及命令类。<br/>
 ![image](https://github.com/riwfnsse/NYear/blob/master/NYear.ODA.DevTool/Images/ORM_Tool.png)

<br/> 第二个是一个简单的 SQL 语句执行工具，可以执行简单的 SQL 语句，对SQLite 等缺少 UI 操作工具的数据库来说是比较实用的。<br/>
 ![image](https://github.com/riwfnsse/NYear/blob/master/NYear.ODA.DevTool/Images/SQL_Tool.png)

<br/> 第三个是数据库复制工具，能够在不同数据库之间复制表数据。<br/>
 ![image](https://github.com/riwfnsse/NYear/blob/master/NYear.ODA.DevTool/Images/DBCopy_Tool.png)



