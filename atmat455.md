# ROTINAS DE LIBERAÇÃO DE ESTOQUE COM CORTES E TRANSFERÊNCIA FILIAIS

### CORTES
1. Para o browse, a exibição deverá ser feita será por pedido e seguindo o filtro padrão da rotina MATA455;
2. Após filtrar quais terão corte/liberação, iniciar o processamento pelo botão em tela;
3. O processamento deverá passar item por item de cada pedido filtrado e analisar a quantidade em estoque de cada item (sempre olhando armazém da SC9 e usando rotina `CalcEst()`) somando a quantidade da Matriz com a quantidade da(s) filial(is) e armazenando numa variável `nEstTotal`;
    1. Se um item não possuir estoque suficiente, exibir uma tela indicando o corte para o usuário. Esta tela terá dois fluxos:
        1. Se o usuário cancelar a tela, <mark>todo o pedido</mark> deverá retornar p/ stand by. Esse tratamento é válido para todos os itens do pedido;
        2. Se todos os itens do pedido forem confirmados, o pedido segue para o processamento do corte;
4. Seguindo para a análise de cortes o sistema deverá ser automático:
    1. Para cada pedido e antes de processar o corte de cada item, a SC9 deverá ser estornada;
    2. Em seguida, cortar a C6_QTDVEN para `C6_QTDVEN – nVlrCorte` sendo o valor da varíavel `nVlrCorte` calculado durante o passo 3. Se `nVlrCorte == C6_QTDVEN` ou se `nEstTotal <= 0`, eliminar o item no pedido.
    3. Para cada pedido, realizar um `MSExecAuto()` fazendo as alterações processadas;
5. Depois de processar todos os cortes, o sistema deverá novamente, para cada item de cada pedido, analisar se o `C6_QTDVEN` pode ser atendido apenas com o saldo da matriz, seguindo para dois fluxos:
    1. Se `C6_QTDVEN` for maior que o estoque disponível na Matriz:
        1. Criar uma varíavel `nQtdFilial` atribuindo `nQtdFilial := C6_QTDVEN - nQtdMatriz`;
        2. Ajustar `SC6->C6_QTDVEN := C6_QTDVEN - nQtdFilial`;
        3. Criar uma nova linha no pedido, para cada produto que utilizará estoque na filial, com `SC6->C6_QTDVEN := nQtdFilial` e `SC6->C6_TES := '9F2'`;
    2. Realizar a liberação de estoque, para cada pedido, utilizando a rotina `MaLibDoFat()` como no exemplo: `nQtdeMalib := MaLibDoFat( SC6->( Recno() ), SC6->C6_QTDVEN, .T., .T., .F., .F., .T., .F., NIL,,, .T. )`;
6. Depois da nova liberação, será necessario criar um pedido de venda "espelhado" na filial, com os itens de TES "9F2" do pedido original;
    1. Cada novo pedido deverá ser liberado após sua criação, utilizando a `MaLibDoFat()` como no exemplo e TES de movimentação de estoque (ex.: '502');
    2. Estes novos pedidos, quando liberados, irão criar Ordens de Separação no WMS e assim o processo automático e o processo da rotina de Liberação de Estoque Com Cortes, se encerra;
---
### TRANSFERÊNCIA FILIAIS
1. Será criada uma nova rotina para faturar e transferir para a matriz todos os pedidos que já tiverem sido separados no WMS. Esta rotina será chamada pelo usuário;
    1. Para o faturamento automático será necessário utilizar a rotina `MaPvlNFs()` montada com os itens da SC9, filtrando apenas os aptos à faturar.
    2. Depois de faturar os pedidos, utilizar a rotina `MaNfs2Nfs()` para preparar o documento de saida de cada pedido faturado anteriormente;
    3. Finalizar com `MSExecAuto` da MATA103 para dar entrada desse documento na Matriz sem movimentar estoque;
