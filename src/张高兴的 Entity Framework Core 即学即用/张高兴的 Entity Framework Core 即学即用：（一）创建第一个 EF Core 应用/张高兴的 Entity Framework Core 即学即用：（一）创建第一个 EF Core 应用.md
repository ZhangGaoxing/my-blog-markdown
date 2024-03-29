## 写在前面

Entity Framework Core (EF Core) 是 .NET 平台流行的对象关系映射（ORM）框架。虽然 .NET 平台中 ORM 框架有很多，比如 Dapper、NHibernate、PetaPoco 等，并且 EF Core 的性能也不是最优的（这是由于 EF 的实体跟踪特性，将其禁用后可以大幅提升性能），但依然吸引到很多后端开发者的使用，原因如下：
1. EF Core 由 .NET 官方进行开发维护，出现问题解决较为及时，这是很多国产 ORM 框架不具有的优势；
2. EF Core 和 C# 语法高度绑定，使用 LINQ 不再需要编写复杂的数据库访问代码；
3. EF Core 支持大部分流行的数据库，切换数据库时只需要更改数据库访问驱动，并不需要更改业务逻辑。
因此在项目中使用 EF Core 不一定是最优的，但一定不会错。

《张高兴的 Entity Framework Core 即学即用》系列博客将会从实践的角度去介绍 EF Core。由于学习的是数据库访问技术，因此还需要一个数据库供我们实践。将根据如下背景设计一个数据库，本系列博客将基于此数据库进行实践：
> 新冠肺炎的流行打破了人们原有的正常生活。为了更好地预防和控制疫情，我们决定开发一个病毒检测管理系统，用于记录公民的核酸检测报告。核酸检测在医院进行，所有检测报告将由病毒检测管理系统收集统计。当前有多家医院可以进行核酸检测，未来这些医院的数量也会增加。考虑到病毒的变异以及未来的扩展性，病毒检测管理系统还需要支持存储不同病毒的检测报告。核酸检测的流程由收集患者的基本信息开始，然后是收集样本的类型，最后出具带有医生姓名的核酸检测报告。

《张高兴的 Entity Framework Core 即学即用》系列博客使用 .NET 6 和 EF Core 6 进行编码，保证了技术的时效性。和绝大部分 EF Core 的教程不同，这里并没有选择使用 SQL Server，而是使用 PostgreSQL 数据库。此处并没有否认 SQL Server 是一款优秀的数据库，并且 EF 的很多特性在 SQL Server 上表现更好，可以说 SQL Server 是 EF Core 的最佳实践。但 SQL Server 最致命的问题是闭源并且收费，现在虽然可以跨平台运行，但这个方向仍有很长的路要走。PostgreSQL 开源且免费，甚至可以运行在 ARM 的 Linux 开发板上，性能也要优于 MySQL。PostgreSQL 扩展性高，拥有庞大的插件群，并且还具有一些“领先时代”的功能，可以说是数据库界的 C#。当然本系列博客并没有涉及到数据库的原生操作，如果你不想使用 PostgreSQL，可以直接将 NuGet 包替换成对应数据库的即可，这也是 EF Core 的优势。

《张高兴的 Entity Framework Core 即学即用》系列博客共分为 4 个部分：
1. 第一部分将从 0 开始创建一个 EF Core 应用，介绍了使用 Database First 的方式以及手工的方式生成实体类，并且尝试查询一张表的数据；
2. 第二部分介绍了 EF Core 的实体状态以及增删改查等数据库操作；
3. 第三部分实现了一个 EF Core 的帮助类，以简化数据库的操作和增强扩展性；
4. 第四部分使用 Razor 引擎实现了一个实体类生成工具。

每一篇博客在介绍功能点时都附带有简单的示例，每一章的最后还附有若干个小练习，希望读者可以借着练习帮助理解，之后根据项目中遇到的问题再学习其他的内容。欢迎批评与指正，有任何的问题都可以通过邮件或者评论的方式与我交流。

<p align="right">张高兴</br>2022年3月22日</p>

---

本文将使用 .NET 6 创建一个控制台程序，从 0 开始，学习 EF Core 的使用。通过本文你可以学到：
1. 使用 Database First 的方式生成实体类；
2. 熟悉实体类中的 EF Core Attribute；
3. 查询一张表的数据；
4. 使用 `Docker` 拉取镜像。

[TOC]

## 准备工作

准备工作包含两部分：安装数据库与创建数据库。EF Core 对 PostgreSQL 的版本没有要求，但后续的博客在介绍编写实体类生成工具时要求 12 及以上的版本。

### 安装 PostgreSQL

#### 直接安装

PostgreSQL 支持在绝大多数操作系统下运行，下载地址：https://www.postgresql.org/download

Windows 下载 exe 安装包，安装时直接点击“下一步”即可，无需额外配置。如果使用 Debian 系列的 Linux 发行版时，直接使用 apt 进行安装，也无需其他操作。其他的操作系统建议根据下载地址中的安装指南进行操作。

#### 使用 Docker 拉取镜像

1. 拉取 PostgreSQL 镜像：
```shell
docker pull postgres
```
2. 创建卷，用于持久化数据库数据：
```shell
docker volume create pgsql_data
```
3. 运行镜像，端口映射为 `54321`，密码配置为弱密码 `@Passw0rd`：
```shell
docker run -d --name pgsql -p 54321:5432 --restart=always -e POSTGRES_PASSWORD='@Passw0rd' -e TZ='Asia/Shanghai' -e ALLOW_IP_RANGE=0.0.0.0/0 -v pgsql_data:/var/lib/postgresql postgres
```

### 数据库的表结构

数据库的设计取决于业务的需求，对同样的业务，每个人的设计都有可能不同，数据库的设计并没有标准答案，读者们或许有更好的设计方案，这里给出的表结构仅供参考，只是为了满足教程的需要：

![](1.jpg)

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e0f2ff;color: #002b4d;margin: 10px">
    <p style="margin-top:0;font-weight: bold">🛈&nbsp;重要</p>
    <div>表结构这里不过多的讲解，虽然使用 EF Core 并不需要掌握复杂的数据库知识，也不需要会写复杂的 SQL，但必要的表结构关系还是应该要理解。</div>
</div>

下面使用熟悉的数据库管理工具，如 pgAdmin、Navicat 等，创建数据库 `pandemic`，具体的执行 SQL 如下，删减了字段注释等不必要的语句：

```sql
create table doctor (
   id                   SERIAL not null,
   name                 VARCHAR(20)          not null,
   hospital_id          INT4                 not null,
   is_deleted           INT4                 not null default 0,
   creator_id           VARCHAR(50)          null,
   created_dt           TIMESTAMP            not null default 'now()',
   modifier_id          VARCHAR(50)          null,
   modified_dt          TIMESTAMP            not null default 'now()',
   constraint PK_DOCTOR primary key (id)
);
create table hospital (
   id                   SERIAL not null,
   name                 VARCHAR(20)          not null,
   is_deleted           INT4                 not null default 0,
   creator_id           VARCHAR(50)          null,
   created_dt           TIMESTAMP            not null default 'now()',
   modifier_id          VARCHAR(50)          null,
   modified_dt          TIMESTAMP            not null default 'now()',
   constraint PK_HOSPITAL primary key (id)
);
create table patient (
   id                   SERIAL not null,
   name                 VARCHAR(20)          not null,
   sex                  VARCHAR(10)          null,
   age                  INT2                 null,
   mobile               VARCHAR(15)          null,
   is_deleted           INT4                 not null default 0,
   creator_id           VARCHAR(50)          null,
   created_dt           TIMESTAMP            not null default 'now()',
   modifier_id          VARCHAR(50)          null,
   modified_dt          TIMESTAMP            not null default 'now()',
   constraint PK_PATIENT primary key (id)
);
create table report (
   id                   SERIAL not null,
   report_type_cd       VARCHAR(20)          not null,
   doctor_id            INT4                 not null,
   patient_id           INT4                 not null,
   result               BOOL                 not null default FALSE,
   collect_time         TIMESTAMP            null,
   test_time            TIMESTAMP            null,
   report_time          TIMESTAMP            null,
   description          VARCHAR(200)         null,
   is_deleted           INT4                 not null default 0,
   creator_id           VARCHAR(50)          null,
   created_dt           TIMESTAMP            not null default 'now()',
   modifier_id          VARCHAR(50)          null,
   modified_dt          TIMESTAMP            not null default 'now()',
   constraint PK_REPORT primary key (id)
);
create table report_type (
   cd                   VARCHAR(20)          not null,
   name                 VARCHAR(20)          null,
   description          VARCHAR(200)         null default 'covid',
   is_deleted           INT4                 not null default 0,
   creator_id           VARCHAR(50)          null,
   created_dt           TIMESTAMP            not null default 'now()',
   modifier_id          VARCHAR(50)          null,
   modified_dt          TIMESTAMP            not null default 'now()',
   constraint PK_REPORT_TYPE primary key (cd)
);

alter table doctor
   add constraint FK_DOCTOR_REFERENCE_HOSPITAL foreign key (hospital_id)
      references hospital (id)
      on delete restrict on update restrict;
alter table report
   add constraint FK_REPORT_REFERENCE_DOCTOR foreign key (doctor_id)
      references doctor (id)
      on delete restrict on update restrict;
alter table report
   add constraint FK_REPORT_REFERENCE_REPORT_T foreign key (report_type_cd)
      references report_type (cd)
      on delete restrict on update restrict;
alter table report
   add constraint FK_REPORT_REFERENCE_PATIENT foreign key (patient_id)
      references patient (id)
      on delete restrict on update restrict;

INSERT INTO report_type (cd, name) VALUES ('COVID-1', '新冠肺炎1号');
INSERT INTO report_type (cd, name) VALUES ('COVID-2', '新冠肺炎2号');
INSERT INTO report_type (cd, name) VALUES ('COVID-3', '新冠肺炎3号');
INSERT INTO report_type (cd, name) VALUES ('COVID-4', '新冠肺炎4号');
INSERT INTO report_type (cd, name) VALUES ('COVID-5', '新冠肺炎5号');
INSERT INTO hospital (name) VALUES ('第一人民医院');
INSERT INTO hospital (name) VALUES ('第二人民医院');
INSERT INTO hospital (name) VALUES ('第三人民医院');
INSERT INTO hospital (name) VALUES ('第四人民医院');
INSERT INTO hospital (name) VALUES ('第五人民医院');
```

## Code First 与 Database First

Code First 和 Database First 算是 EF 中比较有特色的功能。简单来说 Code First 是先编写 C# 实体类，EF 会根据实体类之间的关系创建数据库；Database First 是先设计和创建数据库，EF 根据数据库的表结构生成 C# 实体类。对于不熟悉数据库的开发者来说，Code First 似乎非常方便，不需要掌握数据库的知识也可以使用数据库进行开发。Code First 是被应用于领域驱动设计（Domain Driven Design）中的，由于作者并没有 DDD 的实践，因此无法评判 Code First 的实用性。实际上数据库设计有很多优秀的软件，如 PowerDesigner、Navicat Data Modeler 等，并不需要手动的编写创建数据库的 SQL，因此 Database First 是一种非常高效的方式。而 Code First 中手动编写实体类这一步是不可避免的，在大型项目中数十上百的实体类，这些工作量是不容小觑的。因此本文不会介绍 Code First 的有关操作。

## 创建一个 EF Core 应用

项目地址：https://github.com/ZhangGaoxing/ef-core-demo

### 项目结构

创建一个控制台应用和类库，项目结构如下：

![](2.jpg)

`Pandemic` 为控制台应用，用于实践 EF Core 的有关操作；`Pandemic.Models` 为类库，用于提供数据库上下文和实体类。

### 项目依赖

`Pandemic` 需要引用项目 `Pandemic.Models`：

```xml
<ItemGroup>
  <ProjectReference Include="..\Pandemic.Models\Pandemic.Models.csproj" />
</ItemGroup>
```

`Pandemic.Models` 添加如下 NuGet 包引用：

```xml
<ItemGroup>
  <PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="6.0.3" />
</ItemGroup>
```

### 使用 Scaffold-DbContext 命令生成实体类

接下来使用 Database First 的方式去生成实体类。`Scaffold-DbContext` 顾名思义译为“数据库上下文脚手架”，通过此命令生成实体类和数据库上下文。由于新版本的 .NET 已经不再集成 EF Core Tools 了，因此需要在项目中添加 NuGet 包 `Microsoft.EntityFrameworkCore.Tools`。下面切换到 `Pandemic.Models`，即提供实体类的项目中安装 NuGet 包 `Microsoft.EntityFrameworkCore.Tools`。安装完成后，打开 Visual Studio 中的 **工具 - NuGet 包管理器 - 程序包管理器控制台** 执行以下命令：

1. 切换到项目 `Pandemic.Models`：
   ```shell
   PM> cd .\Pandemic.Models
   ```
2. 运行实体类生成命令 `Scaffold-DbContext`，该命令的详细参数请参考 https://docs.microsoft.com/zh-cn/ef/core/cli/powershell#scaffold-dbcontext ：
   ```shell
   Scaffold-DbContext -Connection "Server=127.0.0.1;Port=54321;Database=pandemic;User Id=postgres;Password=@Passw0rd;" -Provider Npgsql.EntityFrameworkCore.PostgreSQL -Context PandemicContext
   ```
   ![](3.jpg)

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold">💡&nbsp;提示</p>
    <div>执行 Scaffold-DbContext 命令报错时，请将 Pandemic.Models 设为启动项目，并且将“程序包管理器控制台”中的“默认项目”也设置为 Pandemic.Models。</div>
</div>

正常运行没有报错后，实体类与数据库上下文就生成完毕了。

![](4.jpg)

之后打开数据库上下文 `PandemicContext.cs`，会发现其中还是有些许的问题，比如创建时间字段 `created_dt` 的默认值为 PostgreSQL 中的 `now()` 函数，但 EF 并没有将其识别出来：

![](5.jpg)

还需要手动的进行一些调整：

![](6.jpg)

由此可见 Database First 的最佳实践应该使用 SQL Server，这种错误只能希望微软在下一个版本尽快解决。

### 手动创建实体类

下面将手动编写两个实体类 `Hospital.cs` 和 `Doctor.cs`，以熟悉 EF Core Attribute 是如何将数据库表和实体类之间建立联系的。

#### 实体属性

每一个实体类都有一组属性，EF Core 会将实体属性映射到数据库表中的列。

##### 表的映射

对数据库表进行映射，使用 `Table()` Attribute。

```csharp
[Table("hospital")]
public class Hospital { }
```

##### 列的映射

对数据库表中的列进行映射，使用 `Column()` Attribute。

```csharp
[Table("hospital")]
public class Hospital
{
    [Column("id")]
    public int Id { get; set; }
}
```

##### 主键的映射

对数据库表中的主键进行映射，使用 `Key()` Attribute。当主键是自增键时，还需要设置 `DatabaseGenerated()` Attribute。

```csharp
[Table("hospital")]
public class Hospital
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    [Column("id")]
    public int Id { get; set; }
}
```

使用上文介绍的 Attribute 完整的映射医院类 `Hospital.cs` 以及医生类 `Doctor.cs`：

```csharp
[Table("hospital")]
public class Hospital
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    [Column("id")]
    public int Id { get; set; }

    [Column("name")]
    public string Name { get; set; }

    [Column("is_deleted")]
    public int IsDeleted { get; set; } = 0;

    [Column("creator_id")]
    public string CreatorId { get; set; }

    [Column("created_dt")]
    public DateTime CreatedDt { get; set; } = DateTime.Now;

    [Column("modifier_id")]
    public string ModifierId { get; set; }

    [Column("modified_dt")]
    public DateTime ModifiedDt { get; set; } = DateTime.Now;
}
```

```csharp
[Table("doctor")]
public class Doctor
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    [Column("id")]
    public int Id { get; set; }

    [Column("name")]
    public string Name { get; set; }

    [Column("hospital_id")]
    public int HospitalId { get; set; }

    [Column("is_deleted")]
    public int IsDeleted { get; set; } = 0;

    [Column("creator_id")]
    public string CreatorId { get; set; }

    [Column("created_dt")]
    public DateTime CreatedDt { get; set; } = DateTime.Now;

    [Column("modifier_id")]
    public string ModifierId { get; set; }

    [Column("modified_dt")]
    public DateTime ModifiedDt { get; set; } = DateTime.Now;
}
```

#### 导航属性

导航（Navigation）属性是数据库表之间的关系在实体类中的体现。设置好实体类之间的导航属性后，可以通过导航属性轻松的查询到关联实体的数据。在设置导航属性之前，首先需要理清楚医院和医生存在着怎样的数量关系，医生是医院的附属，一家医院下面会有多名医生，因此医院和医生之间是一对多的关系。为了方便理解，下面只保留主键、外键和导航属性。

```csharp
[Table("doctor")]
public class Doctor
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    [Column("id")]
    public int Id { get; set; }

    [Column("hospital_id")]
    public int HospitalId { get; set; }

    [ForeignKey("HospitalId")]
    public virtual Hospital Hospital { get; set; }
}
```

```csharp
[Table("hospital")]
public class Hospital
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    [Column("id")]
    public int Id { get; set; }

    public virtual List<Doctor> Doctors { get; set; }
}
```

数据库表之间通过外键建立数量关系，因此配置导航属性使用 `ForeignKey()` Attribute。

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold">💡&nbsp;提示</p>
    <div>导航属性中的 virtual 关键字不是必须的，当使用懒加载（lazy loading）时才有意义。在任何时候都不建议使用懒加载，这会拖慢数据的查询速度。</div>
</div>

### 创建数据库上下文

数据库上下文（DbContext）是 EF 不可或缺的一部分。DbContext 的实例表示程序与数据库的一个会话（session），通过 DbContext 实例对数据库中的数据进行修改和查询。

为了在程序中访问数据库 `pandemic`，使数据库表与创建的 C# 实体类建立映射，需要创建一个数据库上下文类 `PandemicContext.cs`，该类派生自 `Microsoft.EntityFrameworkCore.DbContext`。

```csharp
public class PandemicContext : DbContext
{
    public PandemicContext() { }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder) { }
}
```

`OnConfiguring()` 方法用于配置数据库的一些设置，比如连接字符串、日志等：

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseNpgsql("Server=127.0.0.1;Port=54321;Database=pandemic;User Id=postgres;Password=@Passw0rd;");
}
```

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #ffdacc;color: #651b01;margin: 10px">
    <p style="margin-top:0;font-weight: bold">⚠️&nbsp;警告</p>
    <div>不建议将密码以明文的方式暴露在程序中，在 .NET 中字符串并不是加密的，密码可能会短暂的出现在内存中，或是对程序的反编译都可能会造成密码的泄露。最优的解决方案是不使用密码进行身份验证，或是通过读取外部配置文件，这样也便于程序的维护。</div>
</div>

`OnModelCreating()` 方法用于配置数据库表与实体类之间的映射。由于数据库表中配置了软删除标记 `is_deleted`，当 `is_deleted = 1` 时认为该条数据是删除的，因此还需要对获取的数据进行过滤，使用 `HasQueryFilter()` 方法：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Hospital>()
        .HasQueryFilter(x => x.IsDeleted == 0);
    modelBuilder.Entity<Doctor>()
        .HasQueryFilter(x => x.IsDeleted == 0);
    modelBuilder.Entity<Report>()
        .HasQueryFilter(x => x.IsDeleted == 0);
    modelBuilder.Entity<ReportType>()
        .HasQueryFilter(x => x.IsDeleted == 0);
    modelBuilder.Entity<Patient>()
        .HasQueryFilter(x => x.IsDeleted == 0);
}
```

数据库上下文中通常还包括每个实体类的 `DbSet<TEntity>` 属性。`DbSet<TEntity>` 是用于修改和查询实体的数据，对 `DbSet<TEntity>` 的 LINQ 查询会转换为对应数据库表的查询。最终的数据库上下文类 `PandemicContext.cs` 如下所示：

```csharp
public class PandemicContext : DbContext
{
    public DbSet<Hospital> Hospitals { get; set; }
    public DbSet<Doctor> Doctors { get; set; }
    public DbSet<Report> Reports { get; set; }
    public DbSet<ReportType> ReportTypes { get; set; }
    public DbSet<Patient> Patients { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseNpgsql("Server=127.0.0.1;Port=54321;Database=pandemic;User Id=postgres;Password=@Passw0rd;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Hospital>()
            .HasQueryFilter(x => x.IsDeleted == 0);
        modelBuilder.Entity<Doctor>()
            .HasQueryFilter(x => x.IsDeleted == 0);
        modelBuilder.Entity<Report>()
            .HasQueryFilter(x => x.IsDeleted == 0);
        modelBuilder.Entity<ReportType>()
            .HasQueryFilter(x => x.IsDeleted == 0);
        modelBuilder.Entity<Patient>()
            .HasQueryFilter(x => x.IsDeleted == 0);
    }
}
```

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold">💡&nbsp;提示</p>
    <div>DbSet&lt;TEntity&gt; 属性并不是必须的，只是为了简化操作，在实例化数据库上下文后，仍然可以使用 Set&lt;TEntity&gt;() 方法获取实体类的 DbSet。</div>
</div>

### 从数据库中查询

将实体类配置完关系以及创建数据库上下文后，就可以通过实例化数据库上下文对数据库进行操作。下面切换到 `Pandemic` 控制台应用查询 `report_type` 这张表中的数据：

```csharp
using Pandemic.Models;
PandemicContext context = new PandemicContext();
var types = context.ReportTypes.ToList();
```

![](7.jpg)

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e0f2ff;color: #002b4d;margin: 10px">
    <p style="margin-top:0;font-weight: bold">✏️&nbsp;练习</p>
    <div>1. 试着编写剩下的实体类；</div>
    <div>2. 比较一下 Database First 生成的实体类和数据库上下文，与手工编写的有何不同；</div>
    <div>3. 不使用 DbSet&lt;TEntity&gt; 属性查询 report_type 表的数据。</div>
</div>
