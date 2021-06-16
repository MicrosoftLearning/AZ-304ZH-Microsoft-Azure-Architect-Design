---
lab:
    title: '6：实现基于 Azure SQL 数据库的应用程序'
    module: '模块 6：设计数据库解决方案'
---

# 实验室：实现基于 Azure SQL 数据库的应用程序
# 学生实验室手册

## 实验室场景

Adatum Corporation 有许多两层的应用程序，它们具有基于 .NET Core 的前端和基于 SQL Server 的后端。Adatum 企业体系结构团队正在探索通过利用 Azure SQL 数据库作为数据层来实现这些应用程序的可能性。考虑到现有 SQL Server 后端的间歇性、不可预测的使用以及对前端应用内置的延迟的相对较高的容忍度，Adatum 正在考虑使用 Azure SQL 数据库的无服务器层。 

无服务器是用于单个 Azure SQL 数据库实例的计算层，它可以根据工作负载需求和每秒使用的计算费用自动扩展计算。无服务器计算层还能够在非活动期间自动暂停数据库（仅对存储计费），并在恢复活动时自动恢复数据库。

Adatum 企业体系结构团队还对评估 Azure SQL 数据库提供的网络级安全性感兴趣，以确保在应用程序必须能够连接的情况下，可以将入站连接恢复到特定 IP 地址范围并从其本地位置进行访问，而无需依靠通过站点到站点 VPN 或 ExpressRoute 的混合连接。 
  
为了实现这些目标，Adatum 体系结构团队将测试基于 Azure SQL 数据库的应用程序，包括：

-  实现无服务器层级的 Azure SQL 数据库

-  实现使用 Azure SQL 数据库作为数据存储的 .NET Core 控制台应用


## 目标
  
完成本实验室后，你将能够：

-  实现无服务器层级的 Azure SQL 数据库

-  配置使用 Azure SQL 数据库作为数据存储的基于 .NET Core 的控制台应用


## 实验室环境
  
Windows Server 管理员凭据

-  用户名： **Student**

-  密码： **Pa55w.rd1234**

预计用时：60 分钟


## 实验室文件

-  无


### 练习 1：实现 Azure SQL 数据库
  
本练习的主要任务如下：

1. 创建 Azure SQL 数据库 

1. 连接并查询 Azure SQL 数据库


#### 任务 1：创建 Azure SQL 数据库 

1. 在实验室计算机上，启动 Web 浏览器，导航至 [Azure 门户](https://portal.azure.com)，然后通过提供你将在本实验室使用的订阅中具有所有者角色的用户帐户凭据进行登录。

1. 在 Azure 门户中，搜索并选择 **“SQL 数据库”**，在 **“SQL 数据库”** 边栏选项卡上，选择 **“+ 添加”**。

1. 在 **“创建 SQL 数据库”** 边栏选项卡的 **“基本”** 选项卡上，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 值 | 
    | --- | --- |
    | 订阅 | 本实验室将使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组的名称 **“az30303a-labRG”** |
    | 数据库名 | **az30303a-db1** | 

1. 在 **“服务器”** 下拉列表正下方，选择 **“新建”**，然后在 **“新建服务器”** 边栏选项卡上，指定以下设置并选择 **“确定”** （将其他设置保留为默认值）：

    | 设置 | 值 | 
    | --- | --- |
    | 服务器名称 | 任何有效的全局唯一名称 | 
    | 服务器管理员登录名 | **sqladmin** |
    | 密码 | **Pa55w.rd1234** |
    | 位置 | 你可以在其中预配 SQL 数据库 的 Azure 区域的名称 |

1. 在 **“计算 + 存储”** 标签旁边，选择 **“配置数据库”** 链接。

1. 在 **“配置”** 边栏选项卡上，选择 **“无服务器”**，查看相应的硬件配置和自动暂停延迟设置， 选中 **“启用自动暂停”** 复选框，然后选择 **“应用”**。

1. **返回** **“创建 SQL 数据库”** 边栏选项卡的“基本”选项卡，选择 **“下一页: 联网”>**。 

1. 在 **“创建 SQL 数据库”** 边栏选项卡的 **“联网”** 选项卡上，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 值 | 
    | --- | --- |
    | 连接方式 | **公共终结点** |  
    | 允许 Azure 服务和资源访问此服务器 | **否** |
    | 添加当前客户端 IP 地址 | **是** |

1. 选择 **“下一步: 安全性 >**。

1. 在 **“创建 SQL 数据库”** 边栏选项卡的 **“安全性”** 选项卡中，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 值 | 
    | --- | --- |
    | 启用 Azure Defender for SQL | **以后再说** |

1. 选择 **“下一步: 其他设置 >”**。 

1. 在 **“创建 SQL 数据库”** 边栏选项卡的 **“其他设置”** 选项卡上，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 值 | 
    | --- | --- |
    | 使用现有数据 | **示例** |

1. 选择 **“查看 + 创建”**，然后选择 **“创建”**。 

    >**注意**： 等待创建 SQL 数据库。预配需要约 2 分钟。 


#### 任务 2：连接并查询 Azure SQL 数据库

1. 在 Azure 门户中，搜索并选择 **“SQL 数据库”**，然后在 **“SQL 数据库”** 边栏选项卡上，选择代表新建的 **“az30303a-db1”**  Azure SQL 数据库的条目。

1. 在“SQL 数据库”边栏选项卡上，选择 **“查询编辑器（预览）”**。

1. 在 **“SQL Server 身份验证”** 部分中的 **“密码”** 文本框中，键入 **“Pa55w.rd1234”**，然后选择 **“确定”**。

1. 在 **“查询编辑器（预览）”** 窗格中，在 **“查询 1”** 选项卡上，输入以下查询并选择 **“运行”**：

    ```SQL
    SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
    FROM SalesLT.ProductCategory pc
    JOIN SalesLT.Product p
    ON pc.productcategoryid = p.productcategoryid;
    ```

1. 查看 **“结果”** 选项卡以验证查询是否已成功完成。


### 练习 2：实现使用 Azure SQL 数据库作为数据存储的 .NET Core 控制台应用
  
本次练习的主要任务如下：

1. 识别 Azure SQL 数据库的 ADO.NET 连接信息

1. 创建并配置 .NET Core 控制台应用

1. 测试 .NET Core 控制台应用

1. 配置 Azure SQL 数据库防火墙

1. 验证 .NET Core 控制台应用的功能

1. 删除实验室中部署的 Azure 资源


#### 任务 1：识别 Azure SQL 数据库的 ADO.NET 连接信息

1. 在 Azure 门户中，在上一个练习中部署的 Azure SQL 数据库边栏选项卡的 **“设置”** 部分，选择 **“连接字符串”**。

1. 在 **“ADO.NET”** 选项卡上，记下用于 SQL 身份验证的 ADO.NET 连接字符串。


#### 任务 2：创建并配置 .NET Core 控制台应用

1. 在 Azure 门户中，通过选择搜索文本框右侧的工具栏图标，打开 **“Cloud Shell”** 窗格。

1. 提示选择 **“Bash”** 或 **“PowerShell”** 时，选择 **“Bash”**。 

    >**备注**： 如果这是你第一次打开 **“Cloud Shell”**，会看到 **“未装载任何存储”** 消息，请选择你在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1. 在“Cloud Shell”窗格中，运行以下命令，以创建一个名为 **“az30303a1”** 的新文件夹，并将其设置为当前目录：

   ```sh
   mkdir az30303a1
   cd az30303a1/
   ```

1. 在“Cloud Shell”窗格中，运行以下命令，以基于桌面模板为基于 .NET Core 的应用创建新的应用项目文件：

   ```sh
   dotnet new console
   ```

1. 在“Cloud Shell”窗格中，使用内置编辑器打开 **“az30303a1.csproj”** 文件，并通过在 `<Project>` 标签之间添加以下 XML 元素来修改该文件： 

   ```xml
   <ItemGroup>
       <PackageReference Include="System.Data.SqlClient" Version="4.6.0" />
   </ItemGroup>
   ```

1. 保存并关闭 **“az30303a1.csproj”** 文件。

1. 在“Cloud Shell”窗格中，使用内置编辑器打开 **Program.cs** 文件并通过将其内容替换为以下代码来对其进行修改： 

   ```cs
   using System;
   using System.Data.SqlClient;
   using System.Text;

   namespace sqltest
   {
       class Program
       {
           static void Main(string[] args)
           {
               try 
               { 
                   SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();
                   builder.ConnectionString="<your_ado_net_connection_string>";

                   using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                   {
                       Console.WriteLine("\nQuery data example:");
                       Console.WriteLine("=========================================\n");
        
                       connection.Open();       
                       StringBuilder sb = new StringBuilder();
                       sb.Append("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName ");
                       sb.Append("FROM [SalesLT].[ProductCategory] pc ");
                       sb.Append("JOIN [SalesLT].[Product] p ");
                       sb.Append("ON pc.productcategoryid = p.productcategoryid;");
                       String sql = sb.ToString();

                       using (SqlCommand command = new SqlCommand(sql, connection))
                       {
                           using (SqlDataReader reader = command.ExecuteReader())
                           {
                               while (reader.Read())
                               {
                                   Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                               }
                           }
                       }                    
                   }
               }
               catch (SqlException e)
               {
                   Console.WriteLine(e.ToString());
               }
               Console.WriteLine("\nDone. Press enter.");
               Console.ReadLine(); 
           }
       }
   }
   ```

1. 使编辑器窗口保持打开状态。 

1. 在 Azure 门户中，在显示 **az30303a-db1** 数据库连接字符串的边栏选项卡上，复制 ADO.NET 连接字符串。 

1. 切换回编辑器窗口并将占位符 `<your_ado_net_connection_string>` 替换为你在上一步中复制的连接字符串的值。

1. 在复制到编辑器窗口的连接字符串中，将占位符 `{your_password}` 替换为 **Pa55w.rd1234**。

1. 保存并关闭 **Program.cs** 文件。


#### 任务 3：测试 .NET Core 控制台应用

1. 在“Cloud Shell”窗格中，运行以下命令，以编译新创建的基于 .NET Core 的控制台应用：

   ```sh
   dotnet restore
   ```

1. 在“Cloud Shell”窗格中，运行以下命令，以执行新创建的基于 .NET Core 的控制台应用：

   ```sh
   dotnet run
   ```

1. 请注意，执行控制台应用将触发错误。 

    >**注意**： 这在预料之中，因为要从分配给运行 Cloud Shell 会话的虚拟机的 IP 地址进行连接，就必须得到明确许可。


#### 任务 4：配置 Azure SQL 数据库防火墙

1. 在“Cloud Shell”窗格中，运行以下命令，以确定运行 Cloud Shell 会话的虚拟机的公共 IP 地址：

   ```sh
   curl -s checkip.dyndns.org | sed -e 's/.*Current IP Address: //' -e 's/<.*$//'
   ```

1. 在 Azure 门户中，在显示 **az30303a-db1** 数据库连接字符串的边栏选项卡上，选择 **“概述”**，然后在工具栏中选择 **“设置服务器防火墙”**。

1. 在 **“防火墙设置”** 边栏选项卡上，设置以下条目并选择 **“保存”**：

    | 设置 | 值 | 
    | --- | --- |
    | 规则名称 | **cloudshell** |
    | 起始 IP | 你之前在此任务中确定的 IP 地址 |
    | 结束 IP | 你之前在此任务中确定的 IP 地址 |

    >**备注**： 显然，这只是用于实验用途，因为在你重启 Cloud Shell 会话后，该 IP 地址将会更改。


#### 任务 5：验证 .NET Core 控制台应用的功能

1. 在“Cloud Shell”窗格中，运行以下命令，以执行新创建的基于 .NET Core 的控制台应用：

   ```sh
   dotnet run
   ```

1. 请注意，控制台应用这次将成功执行，并且返回的结果与 Azure 门户 SQL 数据库边栏选项卡中的查询编辑器中显示的结果相同。 


#### 任务 6：删除实验室中部署的 Azure 资源

1. 在“Cloud Shell”窗格中，运行以下命令以列出你在本练习中创建的资源组：

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv
   ```

    > **备注**： 验证输出结果是否仅包含在本实验室中创建的资源组。在本任务中将删除这个组。

1. 在“Cloud Shell”窗格中运行以下命令，以删除在本实验室中创建的资源组

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 在“Cloud Shell”窗格中，运行以下命令，以删除名为 **“az30303a1”** 的文件夹：

   ```sh
   rm -r ~/az30303a1
   ```

1. 关闭“Cloud Shell”窗格。
