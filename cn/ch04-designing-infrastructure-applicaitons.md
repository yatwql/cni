# 第4章 设计基础设施应用程序

在前一章中，我们了解了代表基础架构以及围绕它的部署工具的各种方法和关注。在本章中，我们将看看如何设计部署和管理基础架构的应用程序。我们听取了前一章的关注，重点关注将基础设施作为软件开放的世界，有时称为基础设施作为应用程序。

在云原生环境中，传统的基础架构运营商需要成为基础架构软件工程师。这仍然是一种新兴的做法，与过去的其他运营角色不同。我们迫切需要开始探索模式和制定标准。

基础架构作为软件的代码和基础架构之间的根本区别在于，软件会不断运行，并会根据协调器模式创建或改变基础架构，我们将在本章后面对其进行解释。此外，基础设施作为软件的新范例是，软件现在与数据存储具有更传统的关系，并公开用于定义所需状态的API。例如，该软件可能会根据数据存储中的需要改变基础架构的表示形式，并且可以很好地管理数据存储本身！希望进行协调的状态更改通过API发送到软件，而不是静态代码回购。

基础设施作为软件方向的第一步是让基础设施运营商意识到他们是软件工程师。我们热烈欢迎您来到这个领域！先前的工具（例如配置管理）有类似的目标来改变基础架构操作员的工作职能，但是操作员通常只学会了如何用窄范围应用编写有限的DSL（即单一节点抽象）。

作为一名基础设施工程师，您的任务不仅是掌握设计，管理和运营基础设施的基础原则，还需要掌握您的专业知识并将其封装成坚如磐石的应用程序。这些应用程序代表了我们将要管理和改变的基础架构。

用于管理基础设施的工程软件不是一件容易的事情。我们有传统应用的所有主要问题和担忧，而且我们正处于一个尴尬的空间。基础设施工程几乎是一个几乎荒谬的任务，即构建软件来部署基础架构，这样就可以在新创建的基础架构之上运行相同的软件，这很尴尬。

首先，我们需要了解这个新领域中工程软件的细微差别。我们将研究在云原生社区中得到验证的模式，以了解在我们的应用程序中编写干净和逻辑代码的重要性。但首先，基础设施从哪里来？

##引导问题

1987年3月22日，周日，Richard M. Stallman发送了一封电子邮件到GCC邮件列表，报告成功编译C编译器：

该编译器在68020上编译正确，最近在vax上进行了编译。它最近在68020上正确编译了Emacs，并且还编译了tex-in-C和Kyoto Common Lisp。但是，它可能仍然有许多错误，我希望你会找到我。

我将离开一个月，所以现在报告的错误将不会被处理。 -  Richard M. Stallman

这是软件历史上的一个重要转折点，因为我们是工程软件引导自己。斯托曼从字面上创建了一个可以自行编译的编译器。即使接受这个陈述作为真理在哲学上可能是困难的。

今天我们正在解决与基础设施相同的问题。工程师必须想出解决方案来解决几乎不可能的系统自举问题，并在运行时生效。

一种方法是手动提供云计算和基础架构应用程序中的第一个基础架构。尽管这种方法确实有效，但它通常伴随着警告，即运营商应该在部署更合适的基础架构后销毁初始引导基础架构。这种方法乏味，难以重复，容易出现人为错误。

解决这个问题的更优雅和云端的本地方法是做出（通常是正确的）假设，试图引导基础架构软件的任何人都有本地机器，我们可以利用这个本地机器。现有机器（您的计算机）可作为第一个部署工具，自动在云中创建基础架构。基础架构就位后，您的本地部署工具可以将其自身部署到新创建的基础架构并持续运行。良好的部署工具可以让你在完成后轻松清理。

在初始基础设施引导问题解决后，我们可以使用基础设施应用程序来引导新的基础设施。现在本地计算机已经被排除在外，现在我们完全运行在云端。

## API

在前面的章节中，我们讨论了表示基础结构的各种方法。在本章中，我们将探讨为基础结构提供API的概念。

当API用软件实现时，它很可能会通过数据结构来完成。因此，根据您使用的编程语言，将API视为类，字典，数组，对象或结构是安全的。

API将是数据值的任意定义，可能是少数字符串，少数整数和布尔值。 API将通过某种类似JSON或YAML的编码进行编码和解码，或者甚至可能存储在数据库中。

对于大多数软件工程师来说，为程序提供可版本化的API是很常见的做法。这允许程序随着时间移动，改变和增长。工程师可以做广告以支持较旧的API版本并提供向后兼容性保证。在作为软件的工程基础设施中，由于这些原因，使用API​​是优选的。

寻找一个API作为基础设施的接口是用户使用基础设施作为软件的许多线索之一。传统上，基础设施作为代码是用户将要管理的基础设施的直接表示，而API可能是被管理的确切底层资源之上的抽象.1

最终，API只是代表基础架构的数据结构。

##世界状况

在作为软件工具的基础设施环境中，世界是我们将要管理的基础设施。因此，世界的状态只是我们的计划对世界的审计表示。

世界的状态最终将回到基础设施的内存中。这些内存中的表示应映射到用于声明基础结构的原始API。审核的API或世界状态通常需要保存。

存储介质（有时称为状态存储）可用于存储新审计的API。介质可以是任何传统存储系统，例如本地文件系统，云对象存储或数据库。如果数据存储在类似文件系统的存储中，那么该工具将很可能以逻辑方式对数据进行编码，以便可以在运行时轻松对数据进行编码和解码。这个常见的编码包括JSON，YAML和TOML。

当您开始设计您的程序时，您可能会想要将您存储的其他数据的特权信息存储起来。这可能是也可能不是最佳实践，具体取决于您的安全要求以及您计划存储数据的位置。

记住存储秘密可能是一个漏洞，这一点很重要。在设计软件来控制堆栈最基本的部分时，安全性至关重要。所以通常值得额外的努力来确保秘密是安全的。

除了存储有关程序和云提供商凭证的元信息之外，工程师还需要存储有关基础架构的信息。重要的是要记住，基础设施将以某种方式呈现，理想情况下，该程序易于解码。记住对系统进行更改不会立即发生，而是随着时间的推移也很重要。

将这些数据存储并轻松访问是设计基础架构管理应用程序的重要部分。仅基础设施定义很可能是系统中最具智慧价值的部分。我们来看一个基本的例子，看看这些数据和程序如何一起工作。

重新审视例4-1至4-4，因为它们被用作本章进一步教训的具体例子。

一个文件系统状态存储示例

想象一下，一个数据存储只是一个名为state的目录。在目录中，将有三个文件：

 -  meta_information.yaml
 -  secrets.yaml
 -  infrastructure.yaml

这个简单的数据存储可以准确地封装需要保留的信息，以便有效管理基础设施。

secrets.yaml和infrastructure.yaml文件存储基础架构的表示形式，meta_information.yaml文件（示例4-1）存储其他重要信息，例如基础架构上次调配时间，调配时间和日志信息。

例4-1. state/meta_information.yaml

```yaml
lastExecution:
  exitCode: 0
  timestamp: 2017-08-01 15:32:11 +00:00
  user: kris
  logFile: /var/log/infra.log
```

第二个文件secrets.yaml保存私人信息，用于在程序执行过程中以任意方式验证（例4-2）。

再次，以这种方式存储秘密可能是不安全的。我们仅以secrets.yaml为例。

例4-2. state/secrets.yaml

```yaml
apiAccessToken: a8233fc28d09a9c27b2e2f
apiSecret: 8a2976744f239eaa9287f83b23309023d
privateKeyPath: ~/.ssh/id_rsa
```

第三个文件infrastructure.yaml将包含API的编码表示形式，包括使用的API版本（示例4-3）。我们可以在这里找到基础架构表示，例如网络和DNS信息，防火墙规则和虚拟机定义。

例4-3. state/infrastructure.yaml

```yaml
location: "San Francisco 2"
name: infra1
dns:
    fqdn: infra.example.com
network:
  cidr: 10.0.0.0/12
  serverPools:
  - bootstrapScript: /opt/infra/bootstrap.sh
    diskSize: large
    workload: medium
    memory: medium
    subnetHostsCount: 256
    firewalls:
      - rules:
          - ingressFromPort: 22
            ingressProtocol: tcp
            ingressSource: 0.0.0.0/0
            ingressToPort: 22
    image: ubuntu-16-04-x64
```

起初infrastructure.yaml文件可能看起来只不过是基础设施代码的一个例子。但是，如果仔细观察，您会发现许多定义的指令都是具体基础架构之上的抽象。例如，subnetHostsCount指令是一个整数值并定义了子网的主机的预定数量。该程序将设法为运营商划分网络中定义的更大的无类别域间路由（CIDR）值。运营商不会声明子网，只需要多少主机。操作员其余部分的软件原因。

程序运行时，它可能会更新API并将新的表示写入数据存储区（在这种情况下，它仅仅是一个文件）。继续我们的subnetHostsCount示例，假设程序确实为我们挑选了一个子网CIDR。新的数据结构可能如例4-4所示。

```yaml
location: "San Francisco 2"
name: infra1
dns:
    fqdn: infra.example.com
network:
  cidr: 10.0.0.0/12
serverPools:
  - bootstrapScript: /opt/infra/bootstrap.sh
    diskSize: large
    workload: medium
    memory: medium
    subnetHostsCount: 256
    assignedSubnetCIDR: 10.0.100.0/24
    firewalls:
      - rules:
          - ingressFromPort: 22
            ingressProtocol: tcp
            ingressSource: 0.0.0.0/0
            ingressToPort: 22
    image: ubuntu-16-04-x64
```

请注意程序如何编写assignedSubnetCIDR指令，而不是操作符。另外，请记住更新API的程序是用户如何以软件方式与基础架构进行交互的标志。

现在，请记住，这只是一个例子，并不一定主张使用抽象计算子网CIDR。不同的用例可能需要在应用程序中进行不同的抽象和实现。关于构建基础架构应用程序的一个美丽而强大的事情是，用户可以以任何他们认为有必要解决他们的问题的方式设计软件。

数据存储（infrastructure.yaml文件）现在可以被认为是软件工程领域的传统数据存储。也就是说，该程序可以对文件进行完全的写入控制。

我们会发现，这会带来风险，但对工程师来说也是一个巨大的胜利。基础架构表示不必存储在文件系统的文件中。相反，它可以存储在任何数据存储中，如传统数据库或键/值存储系统。

为了理解软件如何处理这种新的基础架构表示的复杂性，我们必须理解系统中的两种状态 -  API形式的预期状态，可在infrastructure.yaml文件中找到，并且实际状态可以在现实（或审计）中观察到，或世界的状态。

在这个例子中，软件还没有做任何事情或者采取任何行动，而我们正处于管理时间表的开始。因此，世界的实际状态将是什么都没有，而世界的预期状态将是封装在infrastructure.yaml文件中的任何状态。

##协调模式

协调模式是一种软件模式，可用于管理云原生基础架构。该模式强化了基础设施的两种表现形式 - 第一种是基础设施的实际状态，第二种是基础设施的预期状态。

协调模式将迫使工程师有两个独立的途径忘记这些表示，以及实现一个解决方案，以协调实际状态到预期状态。

调和模式可以被认为是一套四种方法和四种哲学规则：

1.为所有输入和输出使用数据结构。
2.确保数据结构是不可变的。
3.保持资源地图简单。
4.使实际状态符合预期状态。

这些模式的消费者可以依靠这些强大的保证。此外，他们将消费者从实施细节中解放出来。

###规则1：为所有输入和输出使用数据结构

实现协调器模式的方法只能接受和返回数据结构.2结构必须在协调器实现的上下文之外定义，但实现必须知道它。

通过仅接受用于输入的数据结构并将其作为输出返回，消费者可以协调其数据存储中定义的任何结构，而不必担心该对帐如何发生。这也允许在运行时或者程序的不同版本中改变，修改或切换实现。

尽管我们希望尽可能经常遵守第一条规则，但是永远不要将数据结构和代码库紧密结合也非常重要。始终遵守最佳的抽象和分离实践，绝不使用API​​的子集来传递函数或类。

###规则2：确保数据结构不可变

考虑像合同或担保这样的数据结构。在调解器模式的上下文中，实际和期望的结构在运行时设置在内存中。这保证了在调解之前结构是准确的。在协调基础设施的过程中，如果结构发生变化，则必须创建一个具有相同保证的新结构。一个明智的基础设施应用程序将强制数据结构的不变性，即使工程师试图改变数据结构，它也不会工作，或者程序会出错（甚至可能不会编译）。

基础架构应用程序的核心组件是它将映射映射到一组资源的能力。资源是需要运行以满足基础架构要求的单个任务。这些任务中的每一个都将负责以某种方式更改基础架构。

基本示例可能是部署新虚拟机，设置新网络或配置现有虚拟机。这些工作单元中的每一个都将被称为资源。每个数据结构都应映射到一定数量的资源。应用程序负责推理结构并创建资源集。图4-1中显示了API映射到单个资源的示例。

图4-1. 将结构映射到资源的图表

协调器模式演示了一种稳定的方法来处理数据结构，因为它会改变资源。由于调解器模式需要比较资源状态，所以数据结构必须是不可变的。这意味着无论何时需要更新数据结构，都必须创建新的数据结构。

注意基础设施的变化。每次发生突变时，实际的数据结构都是陈旧的。一个聪明的基础设施应用程序将意识到这个问题并相应地处理它。

一个简单的解决方案是在发生突变时更新内存中的数据结构。如果每次突变都更新实际状态，则可以观察对帐过程，因为实际状态会随时间经历一系列更改，直到最终匹配预期状态并且对帐完成。

###规则3：保持资源映射简单

在调解者的幕后，这个模式就是一个实现。一个实现只是一组代码，具有创建，修改和删除基础结构的方法。一个程序可能有很多实现。

每个实现最终都需要将数据结构映射到一组资源。这组资源需要按逻辑方式组合在一起，以便程序可以推断每个资源。

除了创建资源的基本模型之外，您必须非常注意每个资源的依赖关系。许多资源依赖于其他资源，这意味着许多基础设施都依赖于其他部分。例如，在将虚拟机放入网络之前，网络需要存在。

协调器模式规定应该使用用于分组资源的最简单的数据结构。

解决资源映射问题是一个工程决策，每个实现都可能会发生变化。仔细挑选数据结构非常重要，因为从工程角度看，协调器需要稳定和平易近人。

映射数据的两种常见结构是集合和图形。

一组是可以迭代的资源的平面列表。在许多编程语言中，这些被称为列表，集合，数组等。

图形是通过指针链接在一起的顶点的集合。根据编程语言，图的顶点通常是结构或类。顶点通过在顶点某处定义的指针有一个到另一个顶点的链接。图形实现可以通过指针从一个跳到另一个来访问每个顶点。

例4-5是Go编程语言中一个基本顶点的例子。

例4-5. 示例顶点

```go
// Vertex is a data structure that represents a single point on a graph. // A single Vertex can have N number of children vertices or none at all. type Vertex struct {
Name string
    Children []*Vertex
}
```

遍历图的例子可能像迭代遍历每个子元素一样简单。这种遍历有时被称为走图。

例4-6是通过Go中写入的深度优先遍历递归访问图中每个顶点的示例。

例4-6. 深度优先遍历

```go
// recursiveWalk will recursively dig into all children, // and their children accordingly and echo the name of // the vertex currently being visited to STDOUT.
func recursiveWalk(v *Vertex){
fmt.Printf("Currently visiting vertex: %s\n", v.Name) for _, child := range v.Children {
        recursiveWalk(child)
    }
}
```

首先，图的简单实现似乎是解决资源图的合理选择，因为可以通过逻辑方式构建图来处理依赖关系。虽然图表会起作用，但它也会带来风险和复杂性。实施图表来绘制资源的最大风险是在图表中有周期。一个循环是当一个图的一个顶点通过一条以上的路径指向另一个顶点时，这意味着遍历该图是一个无止境的操作。

必要时可以使用图形，但在大多数情况下，协调器模式应该映射一组资源，而不是图形。通过使用一个集合，协调器可以程序化地遍历资源，并提供线性方法来解决映射问题。此外，撤销或删除基础架构的过程与通过反向遍历集合一样简单。

###规则4：使实际状态符合预期状态

在协调模式中提供的保证是用户获得准确的或者错误的内容。这是一个保证消耗调节器的工程师可以信赖的。这一点很重要，因为消费者不必关心验证调解者突变是否是幂等性的并按预期结束。实施最终是为了解决这个问题。有了这种保证，在更复杂的操作中使用协调器模式，如控制器或操作员，现在变得更加简单。

在返回调用代码之前，实现应检查新对帐的实际数据结构是否与最初预期的数据结构匹配。如果没有，它应该是错误的。消费者不应该关心验证API，并且如果出现问题，应该能够相信协调器会发生错误。

由于数据结构是不可变的，并且如果调解器模式不成功，API将会出错，所以我们可以高度信任API。对于复杂的系统，重要的是您能够相信您的软件可以工作或以可预测的方式失败。

##协调模式的方法

根据我们刚刚解释的调解模式的信息和规则，让我们看看这些规则是如何实现的。我们将通过查看实现协调器模式的应用程序所需的方法来执行此操作。

协调器模式的第一种方法是GetActual()。这种方法有时称为审计，用于查询基础设施的实际状态。该方法通过生成资源地图，然后程序地调用每个资源以查看存在什么（如果有的话）。该方法将根据查询更新数据结构，并返回表示实际正在运行的填充数据结构。

一个更简单的方法GetExpected()将从数据存储中读取世界的预期状态。在infrastructure.yaml示例（例4-4）的情况下，GetExpected()将简单地解组这个YAML并将其以内存中的数据结构的形式返回。在这一步没有进行资源审计。

最令人兴奋的方法是Reconcile()方法，其中协调器实现将交给世界的实际状态以及世界的预期状态。

这是调解员模式的意图驱动行为的核心。底层协调器实现将使用在GetActual()中使用的相同资源映射逻辑来定义一组资源。然后协调执行将对这些资源进行操作，独立协调每一个资源。

了解每个资源调节步骤的复杂性非常重要。协调器实现必须以两种方式工作。

首先，从所需和实际状态获取资源属性。接下来，将更改应用到最小的一组属性，以使实际状态与所需的状态匹配。

如果在任何时候这两个基础设施冲突的表示，协调执行必须采取行动并改变基础设施。协调步骤完成后，协调器实施必须创建一个新的表示，然后转到下一个资源。在所有资源调和后，协调器实现将新的数据结构返回给接口的调用者。现在这个新的数据结构准确地代表了世界的实际状态，并应该保证它与原始的实际数据结构相匹配。

协调器模式的最后一个方法是Destroy()方法。 wordDestroy()是故意选择在Delete()上的，因为我们希望工程师意识到该方法应该销毁基础结构，并且从不禁用它。 Destroy()方法的实现很简单。它使用与前面实现方法中定义的资源映射相同的资源映射，但仅对资源进行反向操作。

### Go中的模式示例

例4-7是Go编程语言中四种方法中定义的协调器模式。

如果你不知道Go，别担心。该模式可以很容易地用任何语言实现。我们只使用Go，因为它清楚地定义了每种方法的输入和输出类型。请阅读每种方法的注释，因为它定义了每种方法需要做什么以及何时应该使用。

例4-7. 调解器模式界面

```go
// The reconciler interface below is an example of the reconciler pattern.
// It should be used whenever a user intends on mutating infrastructure based on a // state that might have changed over time.
type Reconciler interface {
    // GetActual takes no arguments for input and returns a populated data
    // structure as well as a possible error. The data structure should
    // contain a complete representation of the infrastructure.
    // This is sometimes called an audit. This method
    // should be used to get a real-time representation of what infrastructure is
    // in existence.
GetActual() (*Api, error)
    // GetExpected takes no arguments for input and returns a populated data
    // structure that represents what infrastructure an operator has declared to
    // exist, as well as a possible error. This is sometimes called expected or
    // intended state. This method should be used to get a real-time representation
    // of what infrastructure an operator intends to be in existence.
GetExpected() (*Api, error)
    // Reconcile takes two arguments.
    // actualApi is a populated data structure that is returned from the GetActual
    // method. expectedApi is a populated data structure that is returned from the
    // GetExpected method. Reconcile will return a populated data structure that is
    // a representation of the new "actual" state, as well as a possible error.
    // By definition, the data structure returned here should match
    // the data structure returned from the GetExpected method. This method is
    // responsible for making changes to infrastructure.
    Reconcile(actualApi, expectedApi *Api) (*Api, error)
    // Destroy takes one argument.
// actualApi is a populated data structure that is returned from the GetActual
// method. Destroy will return a populated data structure that is a
// representation of the new "actual" state, as well as a possible error. By
// definition, the data structure returned here should match
// the data structure returned from the GetExpected method.
Destroy(actualApi *Api) (*Api, error)
}
```

##审计关系

随着时间的推移，我们基础设施的最后一次审计变得陈旧，增加了我们对基础设施的表示不准确的风险。因此，权衡是运营商可以交换审计频率以确定基础设施表示的准确性。

和解是隐含的审计。如果没有任何变化，协调员会发现没有需要做的事情，操作就成为审计，验证我们对基础设施的表示是否准确。

此外，如果在我们的基础架构中碰巧发生了一些变化，协调者将检测到这一变化并尝试纠正它。在完成协调后，基础设施的状态将保证准确。因此，隐含地，我们再次审计了基础设施。

配置管理中的审计和协调器模式

基础设施工程师可能熟悉来自配置管理工具的协调模式，这些工具使用类似的方法来改变操作系统。配置管理工具通过一组资源来管理工程师定义的一组清单或配方。

该工具将对系统采取行动以确保实际状态和所需状态匹配。如果没有更改，则执行简单审计以确保状态匹配。

配置管理与云原生基础架构应用程序不同的原因是，配置管理传统上是抽象的单节点，并且不会创建或管理基础架构资源。

一些配置管理工具正在将其在这个领域的使用扩展到不同程度的成功，但它们仍然属于基础设施类的代码范畴，而不是软件提供的基础架构的双向关系。

轻量级和稳定的协调器实施可以产生强大的结果，并快速协调，从而为操作员提供准确的基础设施表示的信心。

###在控制器中使用Reconciler模式

Orchestration工具（如Kubernetes）为我们提供了一个可以方便地运行应用程序的平台。控制器的想法是为预期状态提供控制回路。 Kubernetes建立在这个基础之上。协调器模式可以很容易地审计和协调由Kubernetes控制的对象。

想象一下在以下步骤中循环将无休止地流经调解器模式：

1.调用GetExpected()并从数据存储中读取基础结构的预期状态。
2.调用GetActual()并从环境中读取以获取基础结构的实际状态。
3.调用Reconcile()并调和状态。

以这种方式实施调节器模式的程序将用作控制器。由于很容易看出控制器本身的程序必须有多小，因此该图案的美丽立即变得明显。

此外，改变基础设施就像改变国有商店一样简单。控制器将在下次调用GetExpected()时读取更改并触发协调。负责基础架构的运营商可以放心，稳定可靠的循环在后台安静地运行，在她的基础架构环境中执行她的意愿。现在，运营商通过管理应用来管理基础设施

>控制回路的目标搜寻行为非常稳定。 Kubernetes已经证明了这一点，我们曾经发现过一些没有被注意到的错误，因为控制回路基本上是稳定的，并且会随着时间的推移而自行修正。
>
>如果您被边缘触发，则会冒着损害您的状态的风险，并且永远无法重新创建状态。如果你是电平触发的模式是非常宽容的，并允许组件的空间不应该像他们应该纠正。这使得Kubernetes工作得很好。
>
> ——Joe Beda，Heptio公司首席技术官

销毁基础设施现在就像通知管制员我们希望摧毁基础设施一样简单。这可以通过多种方式完成。一种方法是让控制器尊重禁用的状态文件。这可以通过从开启到关闭翻转来表示。

另一种方式可能是删除国家的内容。无论操作员如何选择发送Destroy()信号，控制器都准备好调用convenienceDestroy()方法。

##结论

基础设施工程师现在是软件工程师，负责构建先进的高度分布式系统，并且向后开发。他们必须编写管理他们负责的基础设施的软件。

虽然这两个学科之间有许多相似之处，但终身学习工程基础设施管理应用程序的交易。诸如引导基础设施之类的难题不断发展，需要工程师不断学习新事物。还需要维护和优化基础设施，这一定会让工程师长期受雇。

本章为用户提供了强大的模式和基础知识，将不明确的API结构映射为粒度资源。这些资源可以应用到您的本地数据中心，私有云之上或公共云中。

了解这些模式的工作原理对于构建可靠的基础架构管理应用程序至关重要。本章阐述的模式旨在为工程师提供构建声明式基础架构管理应用程序的起点和灵感。

在构建基础设施管理应用程序时，没有正确或错误的答案，只要应用程序遵循Unix哲学：“做一件事。做得很好。“