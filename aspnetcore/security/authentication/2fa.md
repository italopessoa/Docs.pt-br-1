---
title: "Autenticação de dois fatores com o SMS"
author: rick-anderson
description: "Mostra como configurar a autenticação de dois fatores (2FA) com o ASP.NET Core"
keywords: "ASP.NET Core, SMS, autenticação, 2FA, autenticação de dois fatores, autenticação de dois fatores"
ms.author: riande
manager: wpickett
ms.date: 8/15/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/authentication/2fa
ms.openlocfilehash: 77404de2f367cb12ba25433899198b69f9e5a7f2
ms.sourcegitcommit: 9a22c64759a7285ba788a37039bea5fe95f45f21
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/15/2017
---
# <a name="two-factor-authentication-with-sms"></a>Autenticação de dois fatores com o SMS

Por [Rick Anderson](https://twitter.com/RickAndMSFT) e [desenvolvedores Suíça](https://github.com/Swiss-Devs)

Este tutorial aplica-se ao ASP.NET Core apenas 1. x. Consulte [geração de código de QR habilitando para aplicativos de autenticador no ASP.NET Core](xref:security/authentication/identity-enable-qrcodes) para ASP.NET Core 2.0 e posterior.

Este tutorial mostra como configurar a autenticação de dois fatores (2FA) usando o SMS. As instruções são fornecidas para [twilio](https://www.twilio.com/) e [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/), mas você pode usar qualquer outro provedor SMS. Recomendamos que você realize [confirmação de conta e senha de recuperação](accconfirm.md) antes de iniciar este tutorial.

Exibição de [exemplo concluído](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/authentication/2fa/sample/Web2FA). [Como baixar](xref:tutorials/index#how-to-download-a-sample).

## <a name="create-a-new-aspnet-core-project"></a>Criar um novo projeto ASP.NET Core

Criar um novo aplicativo web de ASP.NET Core denominado `Web2FA` com contas de usuário individuais. Siga as instruções em [impondo SSL em um aplicativo do ASP.NET Core](xref:security/enforcing-ssl) para configurar e exigir SSL.

### <a name="create-an-sms-account"></a>Criar uma conta do SMS

Criar uma conta SMS, por exemplo, de [twilio](https://www.twilio.com/) ou [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/). Registre as credenciais de autenticação (para o twilio: accountSid e authToken para ASPSMS: chave de usuário confidenciais e a senha).

#### <a name="figuring-out-sms-provider-credentials"></a>Descobrir as credenciais do provedor de SMS

**Twilio:**  
Na guia Painel de sua conta do Twilio, copie o **SID de conta** e **token de autenticação**.

**ASPSMS:**  
De suas configurações de conta, navegue até **chave de usuário confidenciais** e copie-a junto com seu **senha**.

Mais tarde armazenaremos esses valores com a ferramenta de Gerenciador de segredo nas chaves `SMSAccountIdentification` e `SMSAccountPassword`.

#### <a name="specifying-senderid--originator"></a>Especificando SenderID / originador

**Twilio:**  
Na guia números, copie o Twilio **número de telefone**. 

**ASPSMS:**  
No Menu originadores desbloquear desbloquear originadores de uma ou mais ou escolha um originador alfanumérico (não é suportado por todas as redes). 

Posteriormente, armazenará esse valor com a ferramenta Gerenciador de segredo na chave do `SMSAccountFrom`.


### <a name="provide-credentials-for-the-sms-service"></a>Forneça credenciais para o serviço SMS

Usaremos o [padrão de opções](xref:fundamentals/configuration#options-config-objects) para acessar as configurações de conta e a chave de usuário. 

   * Crie uma classe para obter a chave segura do SMS. Para este exemplo, o `SMSoptions` classe é criada no *Services/SMSoptions.cs* arquivo.

[!code-csharp[Main](2fa/sample/Web2FA/Services/SMSoptions.cs)]

Definir o `SMSAccountIdentification`, `SMSAccountPassword` e `SMSAccountFrom` com o [ferramenta Gerenciador de segredo](xref:security/app-secrets). Por exemplo:

```none
C:/Web2FA/src/WebApp1>dotnet user-secrets set SMSAccountIdentification 12345
info: Successfully saved SMSAccountIdentification = 12345 to the secret store.
```
* Adicione o pacote NuGet do provedor de SMS. Do pacote Manager Console (PMC) executar:

**Twilio:**  
`Install-Package Twilio`

**ASPSMS:**  
`Install-Package ASPSMS`


* Adicione código de *Services/MessageServices.cs* arquivo para habilitar o SMS. Use o Twilio ou seção ASPSMS:


**Twilio:**  
[!code-csharp[Main](2fa/sample/Web2FA/Services/MessageServices_twilio.cs)]

**ASPSMS:**  
[!code-csharp[Main](2fa/sample/Web2FA/Services/MessageServices_ASPSMS.cs)]

### <a name="configure-startup-to-use-smsoptions"></a>Configurar a inicialização para usar`SMSoptions`

Adicionar `SMSoptions` no contêiner de serviço no `ConfigureServices` método o *Startup.cs*:

[!code-csharp[Main](2fa/sample/Web2FA/Startup.cs?name=snippet1&highlight=4)]

### <a name="enable-two-factor-authentication"></a>Habilitar a autenticação de dois fatores

Abra o *Views/Manage/Index.cshtml* o arquivo de exibição Razor e remover caracteres de comentário (portanto, nenhuma marcação é comentário removido).

## <a name="log-in-with-two-factor-authentication"></a>Faça logon com a autenticação de dois fatores

* Execute o aplicativo e registrar um novo usuário

![Exibição de registro de aplicativo da Web aberta no Microsoft Edge](2fa/_static/login2fa1.png)

* Toque em seu nome de usuário que ativa o `Index` método de ação no controlador de gerenciamento. Em seguida, toque o número de telefone **adicionar** link.

![Gerenciar o modo de exibição](2fa/_static/login2fa2.png)

* Adicionar um número de telefone que receberá o código de verificação e toque em **enviar o código de verificação**.

![Adicionar página de número de telefone](2fa/_static/login2fa3.png)

* Você receberá uma mensagem de texto com o código de verificação. Insira-o e toque em **enviar**

![Verifique se a página de número de telefone](2fa/_static/login2fa4.png)

Se você não receber uma mensagem de texto, consulte a página de registro do twilio.

* O modo de gerenciar mostra que o número de telefone foi adicionado com êxito.

![Gerenciar o modo de exibição](2fa/_static/login2fa5.png)

* Toque em **habilitar** para habilitar a autenticação de dois fatores.

![Gerenciar o modo de exibição](2fa/_static/login2fa6.png)

### <a name="test-two-factor-authentication"></a>Autenticação de dois fatores do teste

* Fazer logoff.

* Iniciar sessão.

* A conta de usuário habilitou a autenticação de dois fatores, você precisará fornecer o segundo fator de autenticação. Neste tutorial você ativou a verificação do telefone. Os modelos internos de também permitem que você configurar o email como o segundo fator. Você pode configurar os fatores adicionais de segundo para autenticação, como códigos QR. Toque em **enviar**.

![Enviar o modo de exibição de código de verificação](2fa/_static/login2fa7.png)

* Insira o código que você pode obter na mensagem SMS.

* Clicar no **lembrar este navegador** caixa de seleção isentará a necessidade de usar 2FA para fazer logon ao usar o mesmo dispositivo e o navegador. Habilitando 2FA e clicando em **lembrar este navegador** fornecerá 2FA forte proteção contra usuários mal-intencionados que tentar acessar sua conta, desde que eles não têm acesso ao dispositivo. Você pode fazer isso em qualquer dispositivo privado que você usa regularmente. Definindo **lembrar este navegador**, obter a segurança adicional de 2FA de dispositivos que você não usa regularmente e obter a conveniência de não ter percorrer 2FA no seus próprios dispositivos.

![Verifique se o modo de exibição](2fa/_static/login2fa8.png)

## <a name="account-lockout-for-protecting-against-brute-force-attacks"></a>Bloqueio de conta para proteger contra ataques de força bruta

Recomendamos que você use o bloqueio de conta com 2FA. Depois que um usuário fizer logon (por meio de uma conta local ou social), cada tentativa falha de 2FA é armazenada e se o máximo de tentativas (o padrão é 5) for atingido, o usuário é bloqueado por cinco minutos (você pode definir o bloqueio de tempo com `DefaultAccountLockoutTimeSpan`). A seguir configura a conta seja bloqueada por 10 minutos após 10 tentativas com falha.

[!code-csharp[Main](2fa/sample/Web2FA/Startup.cs?name=snippet2&highlight=13-17)] 
