---
title: "Configuração no núcleo do ASP.NET"
author: rick-anderson
description: "Saiba como usar a API de configuração para configurar um aplicativo ASP.NET Core de várias fontes."
keywords: "ASP.NET Core, configuração, JSON, configuração"
ms.author: riande
manager: wpickett
ms.date: 6/24/2017
ms.topic: article
ms.assetid: b3a5984d-e172-42eb-8a48-547e4acb6806
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/configuration
ms.openlocfilehash: 39e76b14af85de34b8443bf4e04d18d13ad2aa90
ms.sourcegitcommit: fb518f856f31fe53c09196a13309eacb85b37a22
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/08/2017
---
<a name=fundamentals-configuration></a>

  # <a name="configuration-in-aspnet-core"></a>Configuração no núcleo do ASP.NET

[Rick Anderson](https://twitter.com/RickAndMSFT), [marca Michaelis](http://intellitect.com/author/mark-michaelis/), [Steve Smith](http://ardalis.com), e [Daniel Roth](https://github.com/danroth27)

A API de configuração fornece uma maneira de configurar um aplicativo baseado em uma lista de pares nome-valor. Configuração é lida em tempo de execução de várias fontes. Os pares nome-valor podem ser agrupados em uma hierarquia de vários níveis. Há provedores de configuração para:

* Formatos de arquivo (INI, JSON e XML)
* Argumentos de linha de comando
* Variáveis de ambiente
* Objetos do .NET na memória
* Um repositório de usuário criptografado
* [Cofre de chaves do Azure](xref:security/key-vault-configuration)
* Provedores personalizados, que você instala ou criar

Cada valor de configuração é mapeado para uma chave de cadeia de caracteres. Não há suporte de ligação interna ao desserializar as configurações em um personalizado [POCO](https://en.wikipedia.org/wiki/Plain_Old_CLR_Object) objeto (uma classe .NET simple com propriedades).

[Exibir ou baixar o código de exemplo](https://github.com/aspnet/docs/tree/master/aspnetcore/fundamentals/configuration/sample)

## <a name="simple-configuration"></a>Configuração simples

O aplicativo de console a seguir usa o provedor de configuração JSON:

[!code-csharp[Main](configuration/sample/ConfigJson/Program.cs)]

O aplicativo lê e exibe as seguintes configurações:

[!code-json[Main](configuration/sample/ConfigJson/appsettings.json)]

Configuração consiste em uma lista hierárquica de pares nome-valor no qual os nós são separados por dois-pontos. Para recuperar um valor, acessar o `Configuration` indexador com a chave do item correspondente:

```csharp
Console.WriteLine($"option1 = {Configuration["subsection:suboption1"]}");
```

Para trabalhar com matrizes em fontes de configuração formatados em JSON, use um índice de matriz como parte da cadeia de caracteres separada por dois-pontos. O exemplo a seguir obtém o nome do primeiro item na anterior `wizards` matriz:

```csharp
Console.Write($"{Configuration["wizards:0:Name"]}, ");
```

Pares de nome-valor gravados interna em `Configuration` provedores são **não** persistentes, no entanto, você pode criar um provedor personalizado que salva valores. Consulte [provedor de configuração personalizado](xref:fundamentals/configuration#custom-config-providers).

O exemplo anterior usa o indexador de configuração para ler valores. A configuração de acesso fora de `Startup`, use o [padrão de opções](xref:fundamentals/configuration#options-config-objects). O *padrão de opções* mostrado mais adiante neste artigo.

É comum ter configurações diferentes para ambientes diferentes, por exemplo, desenvolvimento, teste e produção. O `CreateDefaultBuilder` método de extensão em um aplicativo do ASP.NET Core 2. x (ou usando `AddJsonFile` e `AddEnvironmentVariables` diretamente em um aplicativo do ASP.NET Core 1. x) adiciona provedores de configuração para ler arquivos JSON e sistema de fontes de configuração:

* *appSettings. JSON*
* * appsettings. \<EnvironmentName >. JSON
* variáveis de ambiente

Consulte [AddJsonFile](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.configuration.jsonconfigurationextensions) para obter uma explicação dos parâmetros. `reloadOnChange`só é suportado no ASP.NET Core 1.1 e superior. 

Fontes de configuração são lidos na ordem em que eles sejam especificados. No código acima, as variáveis de ambiente são lidos pela última. Quaisquer valores de configuração definido através do ambiente substituiria aquelas definidas em dois provedores anteriores.

O ambiente normalmente é definido como um dos `Development`, `Staging`, ou `Production`. Consulte [trabalhando com vários ambientes](environments.md) para obter mais informações.

Considerações de configuração:

* `IOptionsSnapshot`pode recarregar os dados de configuração quando ele é alterado. Use `IOptionsSnapshot` se você precisar recarregar os dados de configuração.  Consulte [IOptionsSnapshot](#ioptionssnapshot) para obter mais informações.
* Chaves de configuração diferenciam maiusculas de minúsculas.
* Uma prática recomendada é especificar variáveis de ambiente por último, para que o ambiente local pode substituir qualquer coisa definidas nos arquivos de configuração implantada.
* **Nunca** armazenar senhas ou outros dados confidenciais no código do provedor de configuração ou nos arquivos de configuração de texto sem formatação. Não usa segredos de produção no desenvolvimento ou ambientes de teste. Em vez disso, especifique segredos fora da árvore de projeto para que eles não puderem ser acidentalmente confirmados para o repositório. Saiba mais sobre [trabalhando com vários ambientes](environments.md) e gerenciando [armazenamento seguro de segredos do aplicativo durante o desenvolvimento](../security/app-secrets.md).
* Se `:` não pode ser usado em variáveis de ambiente em seu sistema, substitua `:` com `__` (sublinhado duplo).

<a name=options-config-objects></a>

## <a name="using-options-and-configuration-objects"></a>Usando opções e objetos de configuração

O padrão de opções usa as classes de opções personalizada para representar um grupo de configurações relacionadas. É recomendável que você crie classes separadas para cada recurso dentro de seu aplicativo. Execute as classes separadas:

* O [princípio de diferenciação de Interface (ISP)](http://deviq.com/interface-segregation-principle/) : Classes dependem apenas as definições de configuração que eles usam.
* [Separação de preocupações](http://deviq.com/separation-of-concerns/) : configurações para diferentes partes do seu aplicativo não são dependentes ou combinado com uma da outra.

A classe de opções deve ser não-abstrato com um construtor público sem parâmetros. Por exemplo:

[!code-csharp[Main](configuration/sample/UsingOptions/Models/MyOptions.cs)]

<a name=options-example></a>

No código a seguir, o provedor de configuração JSON está habilitado. O `MyOptions` classe é adicionada ao contêiner de serviço e associada à configuração.

[!code-csharp[Main](configuration/sample/UsingOptions/Startup.cs?name=snippet1&highlight=8,20-21)]

O seguinte [controlador](../mvc/controllers/index.md) usa [construtor injeção de dependência](xref:fundamentals/dependency-injection#what-is-dependency-injection) na [ `IOptions<TOptions>` ](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.options.ioptions-1) para acessar as configurações:

[!code-csharp[Main](configuration/sample/UsingOptions/Controllers/HomeController.cs?name=snippet1)]

Com o seguinte *appSettings. JSON* arquivo:

[!code-json[Main](configuration/sample/UsingOptions/appsettings1.json)]

O `HomeController.Index` método retornará `option1 = value1_from_json, option2 = 2`.

Aplicativos típicos não associar toda a configuração para um arquivo de opções de único. Posteriormente, mostrarei como usar `GetSection` para vincular a uma seção.

No código a seguir, um segundo `IConfigureOptions<TOptions>` serviço é adicionado ao contêiner de serviço. Ele usa um delegado para configurar a associação com `MyOptions`.

[!code-csharp[Main](configuration/sample/UsingOptions/Startup2.cs?name=snippet1&highlight=9-13)]

Você pode adicionar vários provedores de configuração. Provedores de configuração estão disponíveis em pacotes do NuGet. Elas são aplicadas na ordem que eles são registrados.

Cada chamada para `Configure<TOptions>` adiciona um `IConfigureOptions<TOptions>` serviço ao contêiner de serviço. No exemplo anterior, os valores de `Option1` e `Option2` estão especificados na *appSettings. JSON* –, mas o valor de `Option1` é substituído pelo delegado configurado. 

Quando mais de um serviço de configuração é habilitado, a última fonte de configuração especificado "wins" (define o valor de configuração). No código anterior, o `HomeController.Index` método retornará `option1 = value1_from_action, option2 = 2`.

Quando você associa opções de configuração, cada propriedade no seu tipo de opções está associada a uma chave de configuração do formulário `property[:sub-property:]`. Por exemplo, o `MyOptions.Option1` propriedade está associada à chave `Option1`, que são lidos a partir de `option1` propriedade em *appSettings. JSON*. Um exemplo de subpropriedade é mostrado mais adiante neste artigo.

No código a seguir, um terceiro `IConfigureOptions<TOptions>` serviço é adicionado ao contêiner de serviço. Ele é ligado `MySubOptions` a seção `subsection` do *appSettings. JSON* arquivo:

[!code-csharp[Main](configuration/sample/UsingOptions/Startup3.cs?name=snippet1&highlight=16-17)]

Observação: Esse método de extensão exige o `Microsoft.Extensions.Options.ConfigurationExtensions` pacote NuGet.

Usando o seguinte *appSettings. JSON* arquivo:

[!code-json[Main](configuration/sample/UsingOptions/appsettings.json)]

O `MySubOptions` classe:

[!code-csharp[Main](configuration/sample/UsingOptions/Models/MySubOptions.cs?name=snippet1)]

Com os seguintes `Controller`:

[!code-csharp[Main](configuration/sample/UsingOptions/Controllers/HomeController2.cs?name=snippet1)]

`subOption1 = subvalue1_from_json, subOption2 = 200`é retornado.

Você também pode fornecer opções em um modelo de exibição ou injetar `IOptions<TOptions>` diretamente em um modo de exibição:

[!code-html[Main](configuration/sample/UsingOptions/Views/Home/Index.cshtml?highlight=3-4,16-17,20-21)]

<a name=in-memory-provider></a>

## <a name="ioptionssnapshot"></a>IOptionsSnapshot

*Exige o ASP.NET Core 1.1 ou posterior.*

`IOptionsSnapshot`dá suporte a recarregar os dados de configuração quando o arquivo de configuração foi alterada. Ele tem uma sobrecarga mínima. Usando `IOptionsSnapshot` com `reloadOnChange: true`, as opções são vinculadas a `IConfiguration` e recarregados quando alterado.

O exemplo a seguir demonstra como um novo `IOptionsSnapshot` é criada após *config. JSON* alterações. Solicitações para o servidor retornará o mesmo tempo quando *config. JSON* tem **não** alterado. A primeira solicitação após *config. JSON* alterações mostrará uma nova hora.

[!code-csharp[Main](configuration/sample/IOptionsSnapshot2/Startup.cs?name=snippet1&highlight=1-9,13-18,32,33,52,53)]

A imagem a seguir mostra a saída do servidor:

![mostrando de imagem do navegador "última atualização: 22/11/2016 16:43:00"](configuration/_static/first.png)

Atualizar o navegador não altera o valor de mensagem ou hora exibida (quando *config. JSON* não foi alterado).

Alterar e salvar o *config. JSON* e, em seguida, atualize o navegador:

![mostrando de imagem do navegador "última atualização para e: 22/11/2016 4:53 PM"](configuration/_static/change.png)

## <a name="in-memory-provider-and-binding-to-a-poco-class"></a>Provedor na memória e a associação a uma classe POCO

O exemplo a seguir mostra como usar o provedor na memória e vincular a uma classe:

[!code-csharp[Main](configuration/sample/InMemory/Program.cs)]

Os valores de configuração são retornados como cadeias de caracteres, mas a associação permite que a construção de objetos. Associação permite que você recupere POCO objetos ou gráficos de objeto inteiro até mesmo. O exemplo a seguir mostra como associar a `MyWindow` e usar o padrão de opções com um aplicativo MVC do ASP.NET Core:

[!code-csharp[Main](configuration/sample/WebConfigBind/MyWindow.cs)]

[!code-json[Main](configuration/sample/WebConfigBind/appsettings.json)]

Associar a classe personalizada em `ConfigureServices` quando estiver criando o host:

[!code-csharp[Main](configuration/sample/WebConfigBind/Program.cs?name=snippet1&highlight=3-4)]

Exibir as configurações de `HomeController`:

[!code-csharp[Main](configuration/sample/WebConfigBind/Controllers/HomeController.cs)]

### <a name="getvalue"></a>GetValue

O exemplo a seguir demonstra o [GetValue<T> ](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.configuration.configurationbinder#Microsoft_Extensions_Configuration_ConfigurationBinder_GetValue_Microsoft_Extensions_Configuration_IConfiguration_System_Type_System_String_System_Object_) método de extensão:

[!code-csharp[Main](configuration/sample/InMemoryGetValue/Program.cs?highlight=27-29)]

O ConfigurationBinder `GetValue<T>` método permite que você especifique um valor padrão (80 no exemplo). `GetValue<T>`para cenários simples e não vincular ao seções inteiras. `GetValue<T>`Obtém os valores escalares de `GetSection(key).Value` convertido em um tipo específico.

## <a name="binding-to-an-object-graph"></a>Associando a um gráfico de objeto

Você pode vincular recursivamente para cada objeto em uma classe. Considere o seguinte `AppOptions` classe:

[!code-csharp[Main](configuration/sample/ObjectGraph/AppOptions.cs)]

O exemplo a seguir associa ao `AppOptions` classe:

[!code-csharp[Main](configuration/sample/ObjectGraph/Program.cs?highlight=15-16)]

**ASP.NET Core 1.1** e superior podem usar `Get<T>`, que trabalha com seções inteiras. `Get<T>`pode ser mais convienent que usar `Bind`. O código a seguir mostra como usar `Get<T>` com o exemplo acima:

```csharp
var appConfig = config.GetSection("App").Get<AppOptions>();
```

Usando o seguinte *appSettings. JSON* arquivo:

[!code-json[Main](configuration/sample/ObjectGraph/appsettings.json)]

O programa exibe `Height 11`.

O código a seguir pode ser usado para a unidade a configuração de teste:

```csharp
[Fact]
public void CanBindObjectTree()
{
    var dict = new Dictionary<string, string>
        {
            {"App:Profile:Machine", "Rick"},
            {"App:Connection:Value", "connectionstring"},
            {"App:Window:Height", "11"},
            {"App:Window:Width", "11"}
        };
    var builder = new ConfigurationBuilder();
    builder.AddInMemoryCollection(dict);
    var config = builder.Build();

    var options = new AppOptions();
    config.GetSection("App").Bind(options);

    Assert.Equal("Rick", options.Profile.Machine);
    Assert.Equal(11, options.Window.Height);
    Assert.Equal(11, options.Window.Width);
    Assert.Equal("connectionstring", options.Connection.Value);
}
```

<a name=custom-config-providers></a>

## <a name="basic-sample-of-entity-framework-custom-provider"></a>Exemplo básico do provedor personalizado do Entity Framework

Nesta seção, é criado um provedor de configuração básica que lê os pares nome-valor de um banco de dados usando EF. 

Definir um `ConfigurationValue` entidade para armazenar valores de configuração no banco de dados:

[!code-csharp[Main](configuration/sample/CustomConfigurationProvider/ConfigurationValue.cs)]

Adicionar um `ConfigurationContext` para armazenar e acessar os valores configurados:

[!code-csharp[Main](configuration/sample/CustomConfigurationProvider/ConfigurationContext.cs?name=snippet1)]

Criar uma classe que implementa [IConfigurationSource](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.configuration.iconfigurationsource):

[!code-csharp[Main](configuration/sample/CustomConfigurationProvider/EntityFrameworkConfigurationSource.cs?highlight=7)]

Criar o provedor de configuração personalizado herdando de [ConfigurationProvider](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.configuration.configurationprovider).  O provedor de configuração inicializa o banco de dados quando ela estiver vazia:

[!code-csharp[Main](configuration/sample/CustomConfigurationProvider/EntityFrameworkConfigurationProvider.cs?highlight=9,18-31,38-39)]

Os valores realçados do banco de dados ("value_from_ef_1" e "value_from_ef_2") são exibidos quando o exemplo for executado.

Você pode adicionar um `EFConfigSource` método de extensão para adicionar a fonte de configuração:

[!code-csharp[Main](configuration/sample/CustomConfigurationProvider/EntityFrameworkExtensions.cs?highlight=12)]

O código a seguir mostra como usar personalizado `EFConfigProvider`:

[!code-csharp[Main](configuration/sample/CustomConfigurationProvider/Program.cs?highlight=21-26)]

Observe que o exemplo adiciona personalizado `EFConfigProvider` depois que o provedor JSON, então todas as configurações do banco de dados irá substituir as configurações do *appSettings. JSON* arquivo.

Usando o seguinte *appSettings. JSON* arquivo:

[!code-json[Main](configuration/sample/CustomConfigurationProvider/appsettings.json)]

O seguinte é exibido:

```console
key1=value_from_ef_1
key2=value_from_ef_2
key3=value_from_json_3
```

## <a name="commandline-configuration-provider"></a>Provedor de configuração de linha de comando

O exemplo a seguir habilita o provedor de configuração de linha de comando último:

[!code-csharp[Main](configuration/sample/CommandLine/Program.cs)]

Use o seguinte para passar parâmetros de configuração:

```console
dotnet run /Profile:MachineName=Bob /App:MainWindow:Left=1234
```

Que exibe:

```console
Hello Bob
Left 1234
```

O `GetSwitchMappings` método permite que você use `-` em vez de `/` e ele extrai os prefixos de subchave à esquerda. Por exemplo:

```console
dotnet run -MachineName=Bob -Left=7734
```

Exibe:

```console
Hello Bob
Left 7734
```

Argumentos de linha de comando devem incluir um valor (pode ser nulo). Por exemplo:

```console
dotnet run /Profile:MachineName=
```

É Okey, mas

```console
dotnet run /Profile:MachineName
```

resulta em uma exceção. Uma exceção será lançada se você especificar um prefixo de linha de comando de - ou -- para o qual não há nenhum mapeamento correspondente do comutador.

## <a name="the-webconfig-file"></a>O arquivo Web. config

Um *Web. config* arquivo é necessário quando você hospeda o aplicativo no IIS ou IIS Express. *Web. config* ativa AspNetCoreModule no IIS para iniciar seu aplicativo. As configurações no *Web. config* habilitar AspNetCoreModule no IIS para iniciar seu aplicativo e definir outras configurações do IIS e módulos. Se você estiver usando o Visual Studio e excluir *Web. config*, Visual Studio irá criar um novo.

### <a name="additional-notes"></a>Observações adicionais

* Injeção de dependência (DI) não está definido até depois `ConfigureServices` é invocado.
* O sistema de configuração não está ciente de injeção de dependência.
* `IConfiguration`tem dois especializações:
  * `IConfigurationRoot`Usado para o nó raiz. Pode disparar um recarregamento.
  * `IConfigurationSection`Representa uma seção de valores de configuração. O `GetSection` e `GetChildren` métodos retornam um `IConfigurationSection`.

### <a name="additional-resources"></a>Recursos adicionais

* [Trabalhando com vários ambientes](environments.md)
* [Armazenamento seguro de segredos do aplicativo durante o desenvolvimento](../security/app-secrets.md)
* [Injeção de dependência](dependency-injection.md)
* [Provedor de configuração do Cofre de chaves do Azure](xref:security/key-vault-configuration)
