# Warp Exchange Gateway
Documentação criada em 08/10/2018 com objetivo de expor novas funcionalidades aos clientes que estão já estão integrados e a novos clientes.

  - Versão: 1.2.1


# Disposição Geral do Serviço em Dois Passos

  ### 1. Adicionar à sua pagina de pagamento as chamadas a API da Warp Exchange.
    - Solicitar o endereço para um pagamento.
    - Confirmar que o pagamento foi realizado.
  ### 2. Configurar um endereço em seu servidor web para que a Warp Exchange possa se comunicar com você. 
    - Toda alteração de Status na Warp Exchange será informada ao cliente, para que ele tenha sempre o mesmo Status para todas as transações.
# Endpoint Principal

  - https://api.warpexchange.com (Todos os métodos dispostos a seguir utilizarão esse endpoint)
  - Para utilizar a API, será necessário cadastrar uma URL de Callback. Através dessa URL lhe informaremos as atualizações de suas transações. (vide parte final desse documento)
  - Todas as chamadas serão através de método POST para os endereços.
  - Todas as requisições requerem o Header:
```sh
Exemplo:
Key: Content-Type
Value: application/json
```
  - A chave de API fornecida pela Warp Exchange no site deve ser utilizada em todas as requisições através do Header authorizationToken:
  ```sh
Exemplo:
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA
```



### /getnewaddress
  - Método responsável por retornar um novo endereço a partir do valor e moeda informado na sua requisição. 
  - Esse é o método que deve ser chamado após escolher o preço final no site do merchant. (Pagar com visa, master ou bitcoin? -> pós click no bitcoin).
  - Esse método deverá ser chamado a cada 5 (dois) minutos no caso de permanencia na tela do cliente. Isso é obrigatório. 
  - ALTERADO: (Extornaremos o valor diretamente para a carteira que nos enviou caso seja recebido posterior a 3 (três) minutos. (um minuto de margem)) - Não é possivel tal funcionalidade devido aos sistemas de corretoras, principais wallets que trabalharão conosco, não reconhecerem devoluções a seus clientes. Sempre que um cliente pagar menor valor, será enviado a uma carteira de bitcoins da Warp Exchange, que estará preparada para que o cliente final, através da loja que efetuou a compra, envie o endereço para "reembolso".
  ##### Nova Funcionalidade:
- Split de Pagamento: o campo "split" foi adicionado ao método. Esse campo solicita um "MarketPlaceToken" e um percentual, "PercentOf". O MarketPlaceToken, se refere ao recebedor e o percentual, a quanto irá receber em relação àquela venda.
Para emitir esse MarketPlaceToken, o parceiro do MarketPlace e o próprio deverão acessar a área logada do site e clicar: Perfil > Integração com MarketPlace, e clicar em "Novo Market Place Token".
Parceiros de MarketPlace são obrigatóriamente clientes da Warp Exchange, quais já foram cadastrados e com documentos válidos. Essses poderão realizar o cadastro como Pessoa Física, entretanto, somente poderão ser vinculados a um MarketPlace, não poderão realizar vendas em seu nome por questões legais envolvidas.

###### Observações:
- Esse campo não é o mesmo que o campo "Token" gerado para uso da API. É extremamente importante que não compartilhem o Token da API, somente o MarketPlaceToken.
- Esse campo não é obrigatório, portanto não necessário ser informado, mesmo sem valores. 
- É necessário enviar o próprio percentual aplicado. Ou seja, a soma dos percentuais deverá ser 100.00, do contrário, será retornada uma mensagem com o erro em questão. O objetivo da obrigatoriedade é impossibilitar o cliente de passar valor menor ou maior do que o qual realmente desejava por eventual erro de calculo/programação.
- As transações realizadas via MarketPlace serão exibidas em um relatório específico e não junto ás vendas realizadas sem tal funcionalidade. Para acessa-las, acesse a área logada do site e clique em: Relatórios > Bitcoin (MarketPlace). Somente aparecerão os pagamentos relacionados ao próprio cliente, não aparecendo os percentuais dos envolvidos. Em breve disponibilizaremos ao MarketPlace uma área possibilitando acompanhar os pagamentos e status dos parceiros. É importante que ele compreenda que o Parceiro é cliente da Warp Exchange, podendo ele tratar diretamente com nosso suporte, caso necessite.

Exemplo de Requisição:
  ```sh

Header
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA

Body
{
  "network": "BTC", /*  Usar TESTNET para testes com a rede paralela Testnet */
  "valueInLocalCurrency": 2000.45, /*  Valor em reais da compra a ser realizada. */
  "merchantSystemID": "T3qwuhdq2148T2", /* Identificador do sistema do cliente */
   "split": [
                      { "MarketPlaceToken": '....iIsInR5cCI6Ikp....', "PercentOf": 10.00 },/*Comissão de vendas à um parceiro*/
                      { "MarketPlaceToken": '....odPn3MaBxYf4le....', "PercentOf": 20.00 },/*Cliente dono do MarketPlace*/
                      { "MarketPlaceToken": '....Pn3MaBnR5cCI....', "PercentOf": 70.00 }/*Vendedor/Quem entrega o produto*/
            ]
}
```
Exemplo de Resposta:
  ```sh
{
    "statusCode": 200, //HTTP Status - 200 == ok / 500 == Internal Server Error.
    "body": {
        "walletAddress": "2Mzyn1FGC...........AfkHiA",
        "valueInSatoshi": 533543
    }
}
```
Para facilitar a exibição, utilize a API de QR code do google:
<img src="https://chart.googleapis.com/chart?chs=250x250&cht=qr&chl=bitcoin:' + result.body.walletAddress + '&amount=' + result.body.valueInSatoshi+'"/>

### /gettransactioninformation
  - Método que retorna os detalhes de uma transação, sendo informado o ID do sistema do merchant.
  - Nessa versão de API ainda é necessário informar a chave de API além do Header no Body. Isso mudará em proximas versões.

Exemplo de Requisição:
  ```sh

Header
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA

Body
{
  "merchantSystemID": "T3UQWHD12148T2"
}
```
Exemplo de Resposta:
  ```sh
{
    "statusCode": 200, //HTTP Status - 200 == ok / 500 == Internal Server Error.
    "body": {
        "TipoMoeda": "BTC",
        "MerchantSystemID": "T3qwuhdq2148T2",
        "EnderecoDoCliente": null,
        "DataRecebimento": null,
        "ValorSolicitadoEmMoedaLocal": "200045", /*Multiplicar por 100.*/
        "Notificado": false, /* Caso já tenhamos notificado o merchant em seu endereço de callback cadastrado */
        "TarifaMineracaoEnviada": null,
        "HashDaTransacao": null,
        "ValorRecebidoEmDigital": null, /* Deve ser o mesmo que o "ValorSolicitadoEmDigital" */
        "ValorSolicitadoEmDigital": "7613332",
        "StatusID": 1,
        "Status": "Requested Address"
    }
}
```
### /gettransactions
  - Método que retorna todas as transações e seus respectivos status.
  - Esse método será alterado em breve para que seja enviada a paginação desejada.

Exemplo de Requisição:
  ```sh

Header
Key: authorizationToken
Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA

Body
{}
```
Exemplo de Resposta:
  ```sh
  {
    "statusCode": 200, //HTTP Status - 200 == ok / 500 == Internal Server Error.
    "body": {
        [
             {
                "TipoMoeda": "TESTNET",
                "MerchantSystemID": "QWIUDH1218928812HDKASJ",
                "EnderecoDoCliente": "2NFtZ52w...p78SN8aT6rBX",
                "DataRecebimento": "2018-08-15T12:54:59.350Z",
                "ValorSolicitadoEmMoedaLocal": "33300",
                "Notificado": true,
                "TarifaMineracaoEnviada": "3660",
                "HashDaTransacao": "0370901047435fad4bfc708...5b9b5ed2b11d9163faa2",
                "ValorRecebidoEmDigital": "100000000",
                "ValorSolicitadoEmDigital": "100000000",
                "StatusID": 9,
                "Status": "Confirmed on Warp Wallet" /* Pode entregar o produto */
            },
            {
                "TipoMoeda": "BTC",
                "MerchantSystemID": null,
                "EnderecoDoCliente": null,
                "DataRecebimento": null,
                "ValorSolicitadoEmMoedaLocal": "0",
                "Notificado": false,
                "TarifaMineracaoEnviada": null,
                "HashDaTransacao": null,
                "ValorRecebidoEmDigital": null,
                "ValorSolicitadoEmDigital": "0",
                "StatusID": 11,
                "Status": "Payment not made" /* Nada foi enviado no prazo estabelecido */
            }
        ]
  }
```
### Exemplo de retorno do Callback
  - Post que iremos realizar quando qualquer alteração de status de qualquer transação for efetivada. 
  - Esse método receberá a chave de API em seu Header, sendo essa a maneira de assegurar que fomos nós que enviamos essa requisição. Em caso de dúvidas do retorno do método, consulte o método /gettransactioninformation.

Exemplo de método de callback em C# para receber as atualizações:
  ```sh
        [HttpPost]
        [AllowAnonymous]
        public string MerchantTransactions(FormCollection collect)
        {
            string token = this.HttpContext.Request.Headers["authorizationToken"];
            if (token == "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...P0LVCem91odPn3MaBxYf4leokkxA")
            {
                //Transação válida, entretanto, consulte nossa API, no método GetTransactionInformation
                if(GetTransactionInformation().Status != collect["StatusID"])
                return "Error - Transação não conteém o mesmo Status que o informado no método descritivo. Sinalize esse caso para o Suporte da Warp Exchange"
                //
                string WEResp = "TID: " + collect["TID"];
                WEResp += " StatusID: " + collect["StatusID"];
                WEResp += " StatusMessage: " + collect["StatusMessage"];
                WEResp += " MerchantTokenToUseInCallbacks: " + collect["MerchantTokenToUseInCallbacks"];

                switch (collect["StatusID"])
                {
                    case "2":
                        //Salve no Banco de Dados da sua loja:
                        //Envio Realizado. Aguardando Confirmação. 
                        //O cliente pagou, mas precisamos esperar 6 confirmações para não considerar fraude.
                        break;
                    case "9":
                        //Salve no Banco de Dados da sua loja:
                        //Pagamento Confirmado. Você já pode entregar o produto.
                        break;
                    case "6":
                        //Salve no Banco de Dados da sua loja:
                        //Pagamento em dinheiro na sua conta realizado. Você já pode consultar seu saldo.
                        break;
                    case "12":
                        //Salve no Banco de Dados da sua loja:
                        //Valor pago foi inferior ao necessário. Informe a seu cliente que caso deseje, poderá receber de volta, informando-nos por e-mail os detalhes do erro cometido.
                        break;
                }
                
                return WEResp;
            }
            return "Error - Wrong Status or Token. Token : " + this.HttpContext.Request.Headers["authorizationToken"] + " Status: " + collect["StatusID"];   
        }
```

- Possíveis status para recebimento:
```sh
ID	Status
1	 Pagamento Solicitado
2	 Aguardando Confirmação na Warp
6	 Pago pela Warp ao Cliente
9	 Confirmado na Warp
11	 Pagamento não Realizado
12	 Pago com menor valor*
13	 Pago com valor maior - Iremos lhe pagar e você escolhe como reverter ao cliente
14	 Estornado ao cliente
```
*Valores pagos porém com valor menor ao solicitado serão enviados para uma carteira paralela e aguardarão solicitação de estorno em até 30 dias. Posterior a esse prazo, o valor será enviado a uma instituição de caridade.
