# Padronização dos Retornos de Débitos
Abaixo segue um modelo padrão para os retornos dos débitos conforme a camada.

## Camada de sistema (Detrans e outros parceiros)

O exemplo abaixo deve ser o padrão adotado pelos sistemas que se comunicam diretamente com os parceiros que retornam débitos veiculares.
```json
{
  "dateTimeConsultation": "2024-04-02T07:34:30.203-03:00",
  "uf": "DF",
  "documentOwner": "99813025204",
  "vehiclePlate": "NGN4976",
  "renavam": "00929620410",
  "chassis": "93YBB8B058J922179",
  "yearManufacture": 2008,
  "modelYear": 2009,
  "messages": [
    "LICENCIAMENTO: VEÍC.C/+ 15 MULTAS-PAGAR PELA OPÇÃO MULTAS"
  ],
  "debts": [
    {
      "id": "f0c965c1-2848-4c4c-b972-6cdc763246fb",
      "debitType": "licenciamento",
      "creationDateTime": "2024-01-01T00:00:00",
      "dueDateTime": "2024-10-10T00:00:00",
      "value": 251.25,
      "description": "Licenciamento - 2024",
      "quota": 0,
      "idLinkedDebits": [],
      "idUnlinkedDabts": [],
      "aiip": "R234266837",
      "guide": "123456",
      "bankSlip": {
        "slipNumber": "196893004797",
        "digitableLine": "10497271738680011964589300479709996750000025125",
        "barcode": "10499967500000251257271786800119648930047970",
        "dueDate": "2024-10-10",
        "value": 251.25
      }
    }
  ]
}
```

| Campo | Tipo | Obrigatoriedade | Descrição |
| :--- | :---: | :---: | :--- |
| dateTimeConsultation  |  date-time-only | S  | Data é hora em que a consulta ocorreu  |
| uf|  string | S  | A UF do estado do veículo.   |
| documentOwner|  string | N  | Documento do proprietário do veículo (amenas números). A obrigatoriedade do campo depende do órgão externo trazer ou não essa informação.  |
| vehiclePlate|  string | S  | Placa do veículo sem mascaras  |
| renavam|  string | S  | Renavam do veículo sem mascaras |
| chassis|  string | N  | Chassi do veículo sem mascaras. A obrigatoriedade depende do órgão/parceiro retornar o dado |
| yearManufacture|  int | N  | Ano de fabricação do veículo. A obrigatoriedade depende do órgão/parceiro retornar o dado |
| modelYear|  int | N  | Ano do modelo do veículo. A obrigatoriedade depende do órgão/parceiro retornar o dado |
| messages|  array[string]| S  | Lista com mensagens retornadas pelo parceiro direcionadas ao cliente final da B23. |
| debts|  array[object]| S  | Lista com os débitos encontrados |
| debts[].id| string  | S  | Identificador único desse débito, podendo variar conforme o parceiro, nos casos de omissão, retornar um UUID |
| debts[].debitType| string  | S  | Tipo do débito:<br/>**generico**: Débitos que não forma possíveis identificar o seu tipo<br/>**ipva**: Débitos de IPVA<br/>**dpvat**: Débitos de DPVAT<br/>**multa_renainf**: Alguns parceiros,como Daycoval, separam as multas de multas RENAINF<br/>**multa**: Multas que não sejam RENAINF<br>**licenciamento**: Débitos de licenciamento<br/>**taxa**: Taxas retornandas por alguns parceiros<br>**seguro**: Débitos de seguro quando retornandos pelos parceiros<br/>**transferencia**: Débitos de transferência<br/>**licenciamento_transferencia**: Alguns parceitos agrupam os débitos de licenciamento e transferência em um débito só |
| debts[].creationDateTime| date-time-only  | N  | Data e hora de criação do débito. Sua obrigatoriedade depende do parceiro retornar esse dado e seu formato seguira conforme explicado abaixo:<br/><br/>Nos casos de IPVA, DPVAT e Licenciamento adicionar apenas o ano de referencia. Ex: IPVA 2024 = "2024-01-01T00:00:00", DPVAT 2023 = "2023-01-01T00:00:00"<br/><br/>Nos casos de multas, em que o parceiro retorna a data em que ela ocorreu, passar essa informação nesse parâmetro |
| debts[].dueDateTime| date-time-only  | N  | Data e hora de vencimento do débito (que nem sempre é a data de vencimento do boleto). Sua obrigatoriedade depende do parceiro retornar essa informação.<br/> Nos caso em que retornar apenas a data, passar hora, minuto e segundo como zero. Ex: "2024-03-01T00:00:00" |
| debts[].value| decimal | S  | Valor do débito |
| debts[].description| string | S  | Descrição do débito |
| debts[].quota| int | S  | Alguns débitos veiculares podem ser parcelados pelo parceiro, como: IPVA, DPVA e em alguns casos Licenciamento, esse parâmetro significa a referencia da cota/parcela. <br/><br/>Nos casos de débitos com cota/parcela única ou que não tenham cota/parcela retornar o valor `0` e nos casos de cota única com desconto retornar `-1` |
| debts[].idLinkedDebits| array[string] | S  | Uma lista com os `id`s dos débitos que esse débito está vinculado  |
| debts[].idUnlinkedDabts| array[string] | S  | Uma lista com os `id`s dos débitos que esse débito **não** pode está vinculado  |
| debts[].aiip | string | N  | Identificador do auto de infração junto ao Daycoval |
| debts[].guide | string | N  | Identificador da guia junto ao Daycoval |
| debts[].bankSlip| object | N  | Dados da guia atrelada ao débito. Sua obrigatoriedade está atrelada ao parceiro retornar essa informação |
| debts[].bankSlip.slipNumber| string | N | Número do boleto. Sua obrigatoriedade está atrelada ao parceiro retornar esse dado |
| debts[].bankSlip.digitableLine| string | S | Código do boleto digitada |
| debts[].bankSlip.barcode| string | N | Código do boleto escameado. Sua obrigatoriedade está atrelada ao parceiro retornar esse dado |
| debts[].bankSlip.dueDate| date-only| S | Data de vencimento do boleto. Nos casos em que o parceiro não retornar esse dados, passar a data atual |
| debts[].bankSlip.value | decimal | S | Valor da guia a ser pago |

## Camada de Experiência (Comunicação com o site)

Abaixo temo o a resposta que deverá ser exposta ao _front-end_ nas consulta de débitos:
```json
{
  "transactionId": "9A8B266FF858",
  "pnh": false,
  "vehicles": [
    {
      "messages": [
        "LICENCIAMENTO: VEÍC.C/+ 15 MULTAS-PAGAR PELA OPÇÃO MULTAS"
      ],
      "vehicle":{
        "uf": "DF",
        "plate": "JFK1057",
        "renavamCode": "00123456789",
        "brandModel": "VW/FUSCA 1300"
      },
      "debts": [
        {
          "type": "multa",
          "id": "55b99741-08e9-4a33-864b-169f0dad09b6",
          "value": 123.45,
          "description": "CJ03090480 - TRANSITAR VELOCIDADE SUPERIOR MAX PERMIT ATE 20%",
          "dateOccurrence": "2024-04-04",
          "dueDate": "2024-04-04",
          "quota": 0,
          "idLinkedDebits": [
            "d08f8363-5a8b-44e4-ad4e-d6a72380837d"
          ],
          "idUnlinkedDabts": [
            "d08f8363-5a8b-44e4-ad4e-d6a72380837d"
          ]
        }
      ]
    }
  ] 
}
```

| Campo | Tipo | Obrigatoriedade | Descrição |
| :--- | :---: | :---: | :--- |
| transactionId | string | S | Um identificador do pedido que será compartilhado entre todos os fluxos que o pedido passar e servirá como a forma do cliente consulta seu pedido.<br/>Deve ser um número não muito grande e único, minha sugestão é usar o procedimento [System].[GetNewRandonId] contido no SQL Serve para sua geração. |
| pnh | boolean | S | _Flag_ que indica uma consulta proveniente do PNH |
| vehicles | array[object] | S | Lista com seu veículos e os respectivos débitos. |
| vehicles[].messages | array[string] | S | Lista com mensagens retornadas pelo parceiro direcionadas ao cliente final da B23. |
| vehicles[].vehicle | object | S | Dados relacionados ao veículo. |
| vehicles[].vehicle.uf | string | S | UF do veículo |
| vehicles[].vehicle.plate | string | S | Placa do veículo sem mascara. |
| vehicles[].vehicle.renavamCode| string | S | Código RENAVAM do veículo. |
| vehicles[].vehicle.brandModel | string | N | Marca e modelo do veículo. Deve se seguir a seguinte formatação  "MARCA/MODELO" |
| vehicles[].debts | array[object] | S | lista com os débitos do veículo. |
| vehicles[].debts[].type | string | S | Tipo do débito:<br/>**generico**: Débitos que não forma possíveis identificar o seu tipo<br/>**ipva**: Débitos de IPVA<br/>**dpvat**: Débitos de DPVAT<br/>**multa_renainf**: Alguns parceiros,como Daycoval, separam as multas de multas RENAINF<br/>**multa**: Multas que não sejam RENAINF<br>**licenciamento**: Débitos de licenciamento<br/>**taxa**: Taxas retornandas por alguns parceiros<br>**seguro**: Débitos de seguro quando retornandos pelos parceiros<br/>**transferencia**: Débitos de transferência<br/>**licenciamento_transferencia**: Alguns parceitos agrupam os débitos de licenciamento e transferência em um débito só |
| vehicles[].debts[].id | string | S | Identificador único do débito |
| vehicles[].debts[].value | decimal | S | Valor do débito. |
| vehicles[].debts[].description | string | S | Descrição do débito. |
| vehicles[].debts[].quota| int | S  | Alguns débitos veiculares podem ser parcelados pelo parceiro, como: IPVA, DPVA e em alguns casos Licenciamento, esse parâmetro significa a referencia da cota/parcela. <br/><br/>Nos casos de débitos com cota/parcela única ou que não tenham cota/parcela retornar o valor `0` e nos casos de cota única com desconto retornar `-1` |
| vehicles[].debts[].dateOccurrence| date-only | S | Data de ocorrência do débito. Nos casos de IPVA, DPVAT e Licenciamento, considerar apenas o **ano** |
| vehicles[].debts[].dueDate | date-only | S | Data de vencimento do débito |
| vehicles[].debts[].idLinkedDebits | array[string] | S | Lista dos `id`s que esse débito deve está vinculado. |
| vehicles[].debts[].idUnlinkedDabts| array[string] | S | Lista dos `id`s que esse débito **não** deve está vinculado. |

## Conversão de Campos
| Camada de sistema | Camada de Experiência | Observação |
| :--- | :--- | :--- |
| uf | vehicles[].vehicle.uf |
| vehiclePlate | vehicles[].vehicle.plate |
| renavam | vehicles[].vehicle.renavamCode |
|  | vehicles[].vehicle.brandModel | É possível conseguir esse dado pelo retorno da Software Express na consulta do veículo |
| debts[].debitType | vehicles[].debts[].type |
| debts[].id | vehicles[].debts[].id |
| debts[].bankSlip.value | vehicles[].debts[].value |
| debts[].description | vehicles[].debts[].description |
| debts[].creationDateTime | vehicles[].debts[].dateOccurrence |
| debts[].debitType | vehicles[].debts[].dueDate |
| debts[].quota | vehicles[].debts[].quota |
| debts[].idLinkedDebits | vehicles[].debts[].idLinkedDebits |
| debts[].idUnlinkedDabts | vehicles[].debts[].idUnlinkedDabts |

