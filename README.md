# TUTORIAL PARA ORGANIZAÇÃO DA BASE DE DADOS - ACESSIBILIDADE E CONECTIVIDADE DAS PRINCIPAIS VIAS DE VIÇOSA
Este é um tutorial com o passo-a-passo de todo o procedimento metodológico utilizado para a preparação da base de dados do município de Viçosa e para cálculo da conectividade e acessibilidade de suas principais vias de acordo com o Plano de Mobilidade do município.

O objetivo desse tutorial é explicar de forma detalhada os procedimentos realizados para organizar a base de dados das vias de Viçosa e com isso, será possível entender o processo e replicá-lo para demais bases de dados.

A conectividade será calculada através dos índices **alfa**, **beta** e **gama** compliados por _Lee e Wong (2005)_. A acessibilidade topológica será cálculada através dos três métodos apresentados por _Lee e Wong (2005)_ e um quarto método foi proposto no 1º Estudo da dissertação derivado dos demais e foi nomeado como **Índice de Acessibilidade Topológica Geral (IATG)**.

A acessibilidade ponderada, nomeada como **Índice de Acessibilidade Topológica Ponderada pelos Atributos (IATPA)**, é derivada de IATG através da ponderação dos atributos presentes nas vias em análise (declividade, largura da rua e pavimentação) utilizando Análise de Decisão Multicritério.

A seleção das vias em análise foi feita de acordo com o Plano de Mobilidade do município que cita as ruas a serem utilizadas e as classificam em **Corredores Estratégicos**, **Corredores Primários**, **Corredores Secundários** e **Corredores Locais**. Algumas vias adicionais foram adicionadas para garantir as conexões da malha e foram chamadas de **Conexoes**.

Os softwares utilizados durante o processo foram:
SIG: `QGIS 3.22.7` (com extensão _Verificador de topologias_) e `ArcGIS 10.5`
Gerenciador de banco de dados: `PostgreeSQL 14` (com o pacote de extensões para dados espaciais _PostGis_)
Linguaguem de programação: `Python 3` (juntamente com a IDE _Jupyter Notebook_)

**OBS.: ESSE TUTORIAL EXPLICARÁ TODO O PROCEDIMENTO DE PREPARAÇÃO, ORGANIZAÇÃO E TRANSFORMAÇÃO DOS DADOS E CÁLCULO DA CONECTIVIDADE E ACESSIBILIDADE (IATG e IATPA) PARA AS PRINCIPAIS VIAS DE VIÇOSA. NA DISSERTAÇÃO ANALISOU-SE NOVE MALHAS VIÁRIAS (A ATUAL MALHA VIÁRIA DE VIÇOSA E OITO MALHAS DERIVADAS DE PROPOSTAS DE INTERVENÇÕES NA REDE), PORÉM OS PROCEDIMENTOS PARA O CÁLCULO DE ALFA, BETA, GAMA, IATG E IATPA PARA TODAS ELAS SÃO SEMELHANTES.LOGO, TODOS OS PROCESSOS AQUI EXPLICADOS PARA A _REDE 1_ (PRINCIPAIS VIAS DE VIÇOSA SEM PROPOSTAS DE INTERVENÇÕES) PODEM SER REPLICADO PARA AS DEMAIS.**

Os arquivos em SQL, Python e principais shapefiles utilizados estão disponíveis para download nesta página no GitHub.

## PASSO-A-PASSO:

### 1. ORIGEM DOS DADOS:

Os dados utilizados nesse trabalho foram fornecidos por Queiroz, Vieira e Castro (2021). O primeiro arquivo nomeado como `Rede_Viaria_RSU_2021` trata-se do shapefile contando as ruas de Viçosa e conta com os atributos como **Nome_Rua**, **Tipo_Eixo**, **Largura**, **Pavimento**,  **Bairro**, entre outros. A malha viária de Vieira e Castro (2021) será utilizada para formar a rede de análise.

**Os arquivos `Rede_Viaria_RSU_2021` e `declividade_pontos_cotados_SAAE` estão localizados na pasta `DADOS INICIAIS`.**

### 2. SELEÇÃO DOS ATRIBUTOS UTEIS:

Diversos atributos presentes em `Rede_Viaria_RSU_2021` não serão úteis para esse artigo. Portanto, da tabela de atributos dos dados originais, apenas o **oneway** (que indica se a via é mão única ou mão dupla), **Pavimento** e **Largura** permaneceram e todos os demais atributos foram excluidos através da ferramenta `Editar campos` do software `QGIS`. Dois novos campos foram criados: **nome_rua**, **class_pm**, sendo que o primeiro será utilizado para incluir o nome das vias, o segundo a classificação de acordo com o plano de mobilidade.

### 3. SELEÇÃO DAS VIAS A SEREM UTILIZADAS:
As vias mencionadas no Plano de Mobilidade de Viçosa foram selecionadas utilizando o `Open Street Map` e o `Google Maps` como pano de fundo. Durante esse processo os campos os campos **nome da rua** e **class_pm** foram preenchidos.

### 4. GARANTINDO A INTEGRIDADE TOPOLÓGICA DA REDE:

Para garantir a integridade topológica da rede foi utilizado o complemento `Verificador de topologia` do software `QGIS`. No botão `Configurações`, seleciona-se o arquivo da malha viária e marca-se a opção `não devem ter dangles`, adiciona-se a regra e confirma. Na janela aberta clica-se em `Validar tudo`. Surgirão pontos vermelhos em alguns locais da malha viária e será possível descobrir quais linhas estão desconectadas (esses pontos vermelhos devem existir apenas onde a via realmente está interrompida, como as extremidades).

### 5. OBTENÇÃO DA REDE EM FORMA DE GRAFOS:

A malha viária obtida até então não está na forma de grafos. Uma rede viária na forma de grafos deve ser dividida apenas na junção entre os vértices. Para gerar a malha viária de Viçosa dessa forma utilizou-se o software `ArcGIS`. Primeiramente utilizou-se a ferramenta `Dissolve`para agregar todos elementos em apenas uma feição e para concluir utilizou-se a ferramenta `Explode Multipart Feature` da caixa de ferramentas `Advanced Editing`. Os atributos presentes no arquivo anterior devem ser reciados e novos também foram criados. Por fim, os atributos dessa rede foram: **comp_via** (comprimento total do arco), **nome_rua** (referente aos nomes da ruas que compõe o arco), **comp_nome_rua** (referente ao comprimento das ruas que compõem o arco), **class_pm** (referente a classificação de acordo com o plano de mobilidade), **comp_class_pm** (comprimento da rua referente ao plano de mobilidade), **oneway** (referente se a rua é sentido único ou não), **pavimento** (referente aos tipos de pavimentação que aquel arco possui), **comp_pavimento** (referente a cada comprimento do tipo de pavimentação daquele arco), **largura_media** (refernte a largura média do arco), **declividade_media** (referente a declividade média do arco) e **declividade_media_abs** (referente ao valor absoluto da declividade média daquele arco). Todos os atributos foram deixados em branco para serem prenchidos posteriormente.

### 6. DIVIDINDO A MALHA VIÁRIA (GERADA APÓS O PASSO 4) EM DIVERSAS MALHAS COM SEUS RESPECTIVOS ATRIBUTOS:

Antes de preencher os atributos do arquivo `vias_grafos`, realizou-se um procedimento com a malha obtida do `passo 4`. A partir dessa malha, gerou-se cinco  novos arquivos utilizando a ferramenta Dissolve do software `ArcGIS`, com o objetivo de separar cada atributo da rede em um arquivo diferente.

Na ferramenta mencionada selecionou-se a malha viária (que não está na forma de grafos) com todos atributos e na aba `Dissolve_Fields (optional)` marcou-se um dos atributos desejados para a separação. Na aba `Output Feature Class` atribuiu-se o nome do novo arquivo (de forma que remeta ao atributo selecionado) e executou-se a ferramenta. Esse processo deve ser feito cinco vezes para gerar os arquivos para cada um dos três atributos (**nome_rua**, **class_pm**, **oneway**, **pavimento**, **largura_media**). Esses cinco novos dados foram manipulados pela ferramenta `Explodir linhas` do software `QGIS`, com o objetivo de obter os atributos de cada segmento de reta presente na malha viária. Os cinco arquivos foram nomeados como `vias_atr_nomeruas` (com o nome de rua de cada segmento de reta), `vias_atr_classpm` (com o tipo de classificação de via de acordo com o Plnao de Mobilidade de Viçosa para cada segmento de reta), `vias_atr_oneway` (com o sentido da via para cada segmento de reta), `vias_atr_largura` (com a largura da via para cada segmento de reta) e `vias_atr_pavimento` (com a pavimentação para cada segmento de reta).

**Os cinco arquivos se encontram na pasta `DADOS_INTERMEDIARIOS`**

### 7. GERAÇÃO DO MDE

O MDE de Viçosa foi gerado a partir do arquivo `declividade_pontos_cotados_SAAE`. Para isso, utilizou-se a ferramenta `Interpolação IDW` do software `QGIS`. O novo arquivo foi recortado através da ferramenta `Recortar raster pela extensão...` com o objetivo de reduzir a extensão do MDE de forma que englobe apenas a área de estudo. Apesar desse passo não ser obrigatório, é interessante recortar o arquivo raster com o objetivo de reduzir o seu tamanho e facilitar os processamentos. O arquivo que contem o MDE de Viçosa foi nomeado como `mde_vicosa`.

**O MDE do município se encontra na pasta `DADOS_INTERMEDIARIOS`**

### 7. IMPORTANDO TODOS OS DADOS PARA O BANCO DE DADOS

Todos arquivos gerados foram importados para o banco de dados. O banco usado nessa dissertação possui o nome `dissertacao_artigo1` e possui todas extensões espaciais provenientes do PostGIS.

Os arquivos `vias_grafos`, `vias_atr_nomeruas`, `vias_atr_classpm`, `vias_atr_oneway`, `vias_atr_pavimento` e `vias_atr_largura` foram importados através da ferramenta `Gerenciador BD...` disponível no software `QGis`. O procedimento é feito selecionando de forma individual o arquivo que será importado na aba **Entrada**, em **Esquema** seleciona-se a opção _public_ e em **Tabela** atribuiu-se o nome desejado da tabela (igual ao nome do arquivo em formato shapefile). Isso foi feito para todos os arquivos.

### 8. COMPLETANDO O ARQUIVO VIAS_GRAFOS

Os atributos do arquivo `vias_grafos` estão majoritariamente em branco. Para preenchê-los utilizou-se da lingaguem Python através da IDE `Jupyter Notebook`. O procedimento necessita da instalação, através do prompt de comando, do pacote `psycopg2` (necessário para manipular o banco de dados dentro da linaguem Python). Depois disso é possível executar os comandos em lingaguem Python.

**Todos os scripts explicados estão localizados na pasta `SCRIPTS_DISSERTACAO`. Essa pasta se divide em duas sub-pastas: SCRIPTS_SQL (com os codigos executados em linguagem SQL) e SCRIPTS_PYTHON (com o ambiente virtual utilizado nesse trabalho). Para acessar os Scripts que estão separados individualmente é necessário acessar a pasta projetos que está localizada em SCRIPTS_PYTHON >> av_dissertacao >> projetos.**

#### 8.1. INSTALAÇÃO DO PACOTE psycopg2

Para instalar o pacote psycopg2 é necessário digitar o seguinte comando no **prompt de comandos**:

    pip install psycopg2
    
#### 8.2. SCRIPTS EM PYTHON:

Os comandos descritos a seguir foram todos executados no **Jupyter Notebook**.

#### 8.2.1. IMPORTANDO O PACOTE psycopg2 e math:

    import psycopg2 as pg
    import math
    
#### 8.2.2. CONECTANDO COM O BANCO DE DADOS:

Inseriu-se as configurações do banco de dados. Ele está sendo utilizado no servidor local (host = 'localhost'), o nome do banco de dados é dissertacao_v2 (database = 'dissertacao_v2'), o nome de usuário é postgres (user='postgres') e a senha do banco de dados é admin (password = 'admin').

    con = pg.connect(host='localhost', 
                    database='dissertacao_v2',
                    user='postgres', 
                    password='admin')
    
    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL
    
    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado DISSERTAÇÃO, que possui usuário postgres e senha admin.


#### 8.2.3. VENDO QUANTOS ARCOS A MALHA POSSUI

É necessário descobrir quantos arcos a rede possui. Para isso executa-se os seguintes comandos:

    tabela_grafos = 'vias_grafos' #TABELA COM OS GRAFOS
    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 8.2.4. ATRIBUINDO NOME DAS VIAS PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais nomes de ruas presentes na tabela `vias_atr_nomeruas` estão contidas na linha com id em análise. Esses nomes (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.
    
    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'nome_rua' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_atr_nomeruas' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vnr' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_nome_rua' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS
    
    #ADICIONANDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA
    
    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX
        
        if id_i == 72: #LINHA COM ID 72 ESTÁ COM PROBLEMA NESSE SCRIPT, INCLUIR O NOME MANUALMENTE.
            continue
            
        else:
            
            sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
    
            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])
    
            lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n
    
            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_dados_str = lista_dados_str + dado + '; '
                else:
                    lista_dados_str = lista_dados_str + dado
    
            sql = f'select sum(st_length({abr_dados}.geom)) from {tabela_dados} {abr_dados}, {tabela_grafos} {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados_grafo}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O COMPRIMENTO DE VIA DOATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
    
            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])
    
            lista_comp_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS DE COMPRIMENTOS SERÃO TRANSFORMADAS EM UMA STRING DA SEGUINTE FORMA: COMP_1; COMP_2; COMP_3; ... ; COMP_n
    
            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_comp_str = lista_comp_str + str(dado) + '; '
                else:
                    lista_comp_str = lista_comp_str + str(dado)
    
            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:
    
            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
            print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.
    
            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DE COMP DE CADA ATRIBUTO:
    
            sql = f"update {tabela_grafos} set {atributo_comp_grafo}='{lista_comp_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
            print(f'ID: {id_i} {atributo_comp_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 8.2.5. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

    #ESSA ESTRUTURA DE REPETIÇÃO PARA CONFERIR OS COMPRIMENTOS DE ARCO E SE AS INFORMAÇÕES ADICIONADAS ANTERIORMENTE ESTÃO CONSISTENTES:
    
    list_len = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A QUANTIDADE DE RUAS PRESENTE NA LINHA DO SHP DOS GRAFOS É IGUAL A QUANTIDADE DE COMPRIMENTOS DE RUA
    
    list_comp = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A A SOMA DOS COMPRIMENTOS DE CADA RUA É IGUAL AO COMPRIMENTO TOTAL DA LINHA DO SHP DOS GRAFOS
    
    for id_i in range(1, id_max + 1): #ESTRUTURA DE REPETIÇÃO PARA PERCORRER DO ARCO 1 AO ARCO DE ID MAXIMO
        
        sql = f'select {atributo_dados}, {atributo_comp_grafo}, comp_via from "{tabela_grafos}" where id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS INFORMAÇÕES DE COMPRIMENTO DE VIA, NOME DE RUAS E COMPRIMENTO DE CADA NOME DE RUA DO SHP DOS GRAFOS.
    
        cur.execute(sql) #EXECUTANDO O COMANDO
    
        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
        
        dados_nome = dados_consultados[0][0].split(';') #LISTA COM NOME DAS RUAS DO ARCO
        
        comp_dado = dados_consultados[0][1].split(';') #LISTA COM O COMPRIMENTO DE CADA RUA DA LISTA
        
        comp_total = round(dados_consultados[0][2], 3) #COMPRIMENTO TOTAL DO ARCO
        
        sum_comp = 0 #CRIANDO VARIAVEL PARA RECEBER A SOMA DE CADA COMPRIMENTO DE RUA
        
        for comp in comp_dado: #ADICIONANDO A VARIAVEL ANTERIOR A SOMA DOS COMPRIMENTOS DE RUA
            sum_comp = sum_comp + float(comp)
            
        sum_comp = round(sum_comp, 3) #ARRENDONDANDO PARA 4 CASAS DECIMAIS A SOMA DAS RUAS
        
        len_id = len(dados_nome) == len(comp_dado) #A QUANTIDADE DE RUAS É IGUAL A QUANTIDADE DO COMPRIMENTO DESSAS RUAS?
        
        list_len.append(len_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA
        
        comp_id = sum_comp == comp_total #A SOMA DO COMPRIMENTO DAS RUAS É IGUAL AO COMPRIMENTO DO ARCO?
        
        list_comp.append(comp_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA      
               
        print(f'===---===---===\nLinha de id {id_i}:\n\nTem a mesma quantidade de item em comp_ruas e nome_ruas?\n{len_id}\n\nComprimento total é igual ao somatorio de cada rua?\n{comp_id}\n\nSoma das ruas:{sum_comp}\nComprimento total do arco: {comp_total}\nDiferença: {comp_total - sum_comp}\n===---===---===\n') #MENSAGEM COM AS COMPARAÇÕES PARA CONFERIR A CONSISTENCIA ENTRE AS INFORMAÇÕES

#### 8.2.5.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 8.2.5.2. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

     list_comp.count(False)
     
#### 8.2.6. ATRIBUINDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará quais classificações de acordo com o Plano de Mobilidade de Viçosa presentes na tabela `vias_atr_classpm` estão contidas na linha com id em análise. Essas classificações (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'class_pm' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_atr_classpm' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vpm' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'class_pm' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_class_pm' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS
    
    #ADICIONANDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA
    
    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX
        
        if id_i == 72: #LINHA COM ID 72 COM PROBLEMA, INSERIR DADOS MANUALMENTE.
            continue
        
        else:
    
            sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
    
            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])
    
            lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n
    
            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_dados_str = lista_dados_str + dado + '; '
                else:
                    lista_dados_str = lista_dados_str + dado
    
            sql = f'select sum(st_length({abr_dados}.geom)) from {tabela_dados} {abr_dados}, {tabela_grafos} {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados_grafo}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O COMPRIMENTO DE VIA DOATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
    
            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])
    
            lista_comp_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS DE COMPRIMENTOS SERÃO TRANSFORMADAS EM UMA STRING DA SEGUINTE FORMA: COMP_1; COMP_2; COMP_3; ... ; COMP_n
    
            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_comp_str = lista_comp_str + str(dado) + '; '
                else:
                    lista_comp_str = lista_comp_str + str(dado)
    
            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:
    
            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
            print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.
    
            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DE COMP DE CADA ATRIBUTO:
    
            sql = f"update {tabela_grafos} set {atributo_comp_grafo}='{lista_comp_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
            print(f'ID: {id_i} {atributo_comp_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 8.2.7. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

    #ESSA ESTRUTURA DE REPETIÇÃO PARA CONFERIR OS COMPRIMENTOS DE ARCO E SE AS INFORMAÇÕES ADICIONADAS ANTERIORMENTE ESTÃO CONSISTENTES:
    
    list_len = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A QUANTIDADE DE RUAS PRESENTE NA LINHA DO SHP DOS GRAFOS É IGUAL A QUANTIDADE DE COMPRIMENTOS DE RUA
    
    list_comp = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A A SOMA DOS COMPRIMENTOS DE CADA RUA É IGUAL AO COMPRIMENTO TOTAL DA LINHA DO SHP DOS GRAFOS
    
    for id_i in range(1, id_max + 1): #ESTRUTURA DE REPETIÇÃO PARA PERCORRER DO ARCO 1 AO ARCO DE ID MAXIMO
        
        sql = f'select {atributo_dados}, {atributo_comp_grafo}, comp_via from "{tabela_grafos}" where id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS INFORMAÇÕES DE COMPRIMENTO DE VIA, NOME DE RUAS E COMPRIMENTO DE CADA NOME DE RUA DO SHP DOS GRAFOS.
    
        cur.execute(sql) #EXECUTANDO O COMANDO
    
        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
        
        dados_nome = dados_consultados[0][0].split(';') #LISTA COM NOME DAS RUAS DO ARCO
        
        comp_dado = dados_consultados[0][1].split(';') #LISTA COM O COMPRIMENTO DE CADA RUA DA LISTA
        
        comp_total = round(dados_consultados[0][2], 3) #COMPRIMENTO TOTAL DO ARCO
        
        sum_comp = 0 #CRIANDO VARIAVEL PARA RECEBER A SOMA DE CADA COMPRIMENTO DE RUA
        
        for comp in comp_dado: #ADICIONANDO A VARIAVEL ANTERIOR A SOMA DOS COMPRIMENTOS DE RUA
            sum_comp = sum_comp + float(comp)
            
        sum_comp = round(sum_comp, 3) #ARRENDONDANDO PARA 4 CASAS DECIMAIS A SOMA DAS RUAS
        
        len_id = len(dados_nome) == len(comp_dado) #A QUANTIDADE DE RUAS É IGUAL A QUANTIDADE DO COMPRIMENTO DESSAS RUAS?
        
        list_len.append(len_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA
        
        comp_id = sum_comp == comp_total #A SOMA DO COMPRIMENTO DAS RUAS É IGUAL AO COMPRIMENTO DO ARCO?
        
        list_comp.append(comp_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA      
               
        print(f'===---===---===\nLinha de id {id_i}:\n\nTem a mesma quantidade de item em comp_tipo_pm e tipo_pm?\n{len_id}\n\nComprimento total é igual ao somatorio de cada rua?\n{comp_id}\n\nSoma das ruas:{sum_comp}\nComprimento total do arco: {comp_total}\nDiferença: {comp_total - sum_comp}\n===---===---===\n') #MENSAGEM COM AS COMPARAÇÕES PARA CONFERIR A CONSISTENCIA ENTRE AS INFORMAÇÕES
       

#### 8.2.7.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 8.2.7.2. QUANTIDADE DE FALTOS (SOMA DOS COMPRIMENTOS)

    list_comp.count(False)

#### 8.2.8. ATRIBUINDO O SENTIDO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual sentido da via presente na tabela `vias_atr_oneway` estão contidas na linha com id em análise. Esse sentido de via (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

OBS.: Caso a linha possua mais de um sentido de via significa que existe um erro na geração da rede viária e isso deve ser consertado manualmente.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'oneway' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_atr_oneway' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vow' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'oneway' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    
    #ADICIONANDO SENTIDO DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA
    
    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX
    
        sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
        
        cur.execute(sql) #EXECUTANDO O COMANDO
        
        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
        
        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
        
        for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
            if dado[0] not in lista_dados:
                lista_dados.append(dado[0])
        
        lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n
        
        for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
            if dado != lista_dados[len(lista_dados) - 1]:
                lista_dados_str = lista_dados_str + dado + '; '
            else:
                lista_dados_str = lista_dados_str + dado
    
        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:
        
        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
        print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 8.2.9. ATRIBUINDO A PAVIMENTAÇÃO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual a pavimentação presente na tabela `vias_atr_pavimento` estão contidas na linha com id em análise. Esse sentido de via (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'pavimento' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_atr_pavimento' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vp' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'pavimento' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_pavimento' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS
    
    #ADICIONANDO O NOME DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA
    
    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX
        
        if id_i == 72: #LINHA DE ID 72 COM PROBLEMAS, COLOCAR O VALOR MANUALMENTE.
            continue
            
        else:
            
            sql = f'select {abr_dados}.{atributo_dados} from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
    
            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])
    
            lista_dados_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS SERÃO TRANSFORMADA EM UMA STRING DA SEGUINTE FORMA: ATRIBUTO_1; ATRIBUTO_2; ATRIBUTO_3; ... ; ATRIBUTO_n
    
            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_dados_str = lista_dados_str + dado + '; '
                else:
                    lista_dados_str = lista_dados_str + dado
    
            sql = f'select sum(st_length({abr_dados}.geom)) from {tabela_dados} {abr_dados}, {tabela_grafos} {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i} group by {abr_dados}.{atributo_dados_grafo}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O COMPRIMENTO DE VIA DOATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
            lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
    
            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NA LISTA CRIADA
                if dado[0] not in lista_dados:
                    lista_dados.append(dado[0])
    
            lista_comp_str = '' #CRIANDO UMA VARIAVEL NO QUAL A LISTA DE DADOS DE COMPRIMENTOS SERÃO TRANSFORMADAS EM UMA STRING DA SEGUINTE FORMA: COMP_1; COMP_2; COMP_3; ... ; COMP_n
    
            for dado in lista_dados: #TRANSFORMANDO A LISTA DE DADOS PARA UMA STRING
                if dado != lista_dados[len(lista_dados) - 1]:
                    lista_comp_str = lista_comp_str + str(dado) + '; '
                else:
                    lista_comp_str = lista_comp_str + str(dado)
    
            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:
    
            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{lista_dados_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
            print(f'ID: {id_i} {atributo_dados_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.
    
            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DE COMP DE CADA ATRIBUTO:
    
            sql = f"update {tabela_grafos} set {atributo_comp_grafo}='{lista_comp_str}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
            print(f'ID: {id_i} {atributo_comp_grafo} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.

#### 8.2.10. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

    #ESSA ESTRUTURA DE REPETIÇÃO PARA CONFERIR OS COMPRIMENTOS DE ARCO E SE AS INFORMAÇÕES ADICIONADAS ANTERIORMENTE ESTÃO CONSISTENTES:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'pavimento' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_atr_pavimento' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vp' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'pavimento' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    atributo_comp_grafo = 'comp_pavimento' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS COMPRIMENTOS DOS NOMES DAS RUAS NA TABELA DO SHP DOS GRAFOS
    
    list_len = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A QUANTIDADE DE RUAS PRESENTE NA LINHA DO SHP DOS GRAFOS É IGUAL A QUANTIDADE DE COMPRIMENTOS DE RUA
    
    list_comp = [] #CRIANDO UMA LISTA EM BRANCO PARA RECEBER OS VALORES DE TRUE OU FALSE REFERENTE SE A A SOMA DOS COMPRIMENTOS DE CADA RUA É IGUAL AO COMPRIMENTO TOTAL DA LINHA DO SHP DOS GRAFOS
    
    for id_i in range(1, id_max + 1): #ESTRUTURA DE REPETIÇÃO PARA PERCORRER DO ARCO 1 AO ARCO DE ID MAXIMO
        
        sql = f'select {atributo_dados}, {atributo_comp_grafo}, comp_via from "{tabela_grafos}" where id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS INFORMAÇÕES DE COMPRIMENTO DE VIA, NOME DE RUAS E COMPRIMENTO DE CADA NOME DE RUA DO SHP DOS GRAFOS.
    
        cur.execute(sql) #EXECUTANDO O COMANDO
    
        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
        
        dados_nome = dados_consultados[0][0].split(';') #LISTA COM NOME DAS RUAS DO ARCO
        
        comp_dado = dados_consultados[0][1].split(';') #LISTA COM O COMPRIMENTO DE CADA RUA DA LISTA
        
        comp_total = round(dados_consultados[0][2], 3) #COMPRIMENTO TOTAL DO ARCO
        
        sum_comp = 0 #CRIANDO VARIAVEL PARA RECEBER A SOMA DE CADA COMPRIMENTO DE RUA
        
        for comp in comp_dado: #ADICIONANDO A VARIAVEL ANTERIOR A SOMA DOS COMPRIMENTOS DE RUA
            sum_comp = sum_comp + float(comp)
            
        sum_comp = round(sum_comp, 3) #ARRENDONDANDO PARA 4 CASAS DECIMAIS A SOMA DAS RUAS
        
        len_id = len(dados_nome) == len(comp_dado) #A QUANTIDADE DE RUAS É IGUAL A QUANTIDADE DO COMPRIMENTO DESSAS RUAS?
        
        list_len.append(len_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA
        
        comp_id = sum_comp == comp_total #A SOMA DO COMPRIMENTO DAS RUAS É IGUAL AO COMPRIMENTO DO ARCO?
        
        list_comp.append(comp_id) #ADICIONANDO A VARIAVEL ANTERIOR A LISTA      
               
        print(f'===---===---===\nLinha de id {id_i}:\n\nTem a mesma quantidade de item em comp_ruas e nome_ruas?\n{len_id}\n\nComprimento total é igual ao somatorio de cada rua?\n{comp_id}\n\nSoma das ruas:{sum_comp}\nComprimento total do arco: {comp_total}\nDiferença: {comp_total - sum_comp}\n===---===---===\n') #MENSAGEM COM AS COMPARAÇÕES PARA CONFERIR A CONSISTENCIA ENTRE AS INFORMAÇÕES
       

#### 8.2.10.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 8.2.10.2. QUANTIDADE DE FALTOS (SOMA DOS COMPRIMENTOS)

    list_comp.count(False)

#### 8.2.11. ATRIBUINDO A LARGURA DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual a largura presente na tabela `vias_atr_largura` estão contidas na linha com id em análise. Esse sentido de via (caso seja mais de um será separado por ponto e vírgurla), serão atualizados na tabela `vias_grafos` logo em seguida.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'largura_media' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE NOME DAS RUAS NA TABELA DO SHP DOS GRAFOS
    tabela_dados = 'vias_atr_largura' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'vl' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    atributo_dados = 'largura' #NOME DA COLUNA QUE CONTEM AS INFORMAÇÕES DOS NOMES DAS RUAS NA TABELA DO SHP DOS NOMES DAS RUAS DA MALHA
    
    #ADICIONANDO A LARGURA DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA
    
    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX
    
        if id_i == 72: #ID 72 COM PROBLEMA, PULAR E INSERIR MANUALMENTE
            continue
        
        else:
            sql = f'select {abr_dados}.{atributo_dados}, st_length({abr_dados}.geom) from "{tabela_dados}" {abr_dados}, "{tabela_grafos}" {abr_grafos} where st_contains({abr_grafos}.geom, {abr_dados}.geom) and {abr_grafos}.id = {id_i}' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O ATRIBUTOS INDICADO DA TABELA DOS SHP DOS ATRIBUTOS, ONDE A GEOMETRIA DOS ELEMENTOS DA TABELA DE ATRIBUTOS ESTÁ CONTIDA NA GEOMETRIA DA TABELA DO GRAFO, PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
            numerador = 0 #CRIANDO UMA LISTA VAZIA, NO QUAL SERÁ SOMADO TODOS OS ATRIBUTOS PONDERADOS PELO TAMANHO DE SUA LINHA
            denominador = 0 #SOMA DOS PESOS
    
            for dado in dados_consultados: #ADICIONANDO OS RESULTADOS NAS LISTAS CRIADAS
                numerador = numerador + float(dado[0])*float(dado[1])
                denominador = denominador + float(dado [1])
    
            largura_me_i = float(f'{numerador/denominador:.2f}') #CALCULANDO A LARGURA MEDIA DO GRAFO i
    
            #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:
    
            sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{largura_me_i}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
            cur.execute(sql) #EXECUTANDO O COMANDO
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
            print(f'ID: {id_i} {largura_me_i} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.


#### 8.2.12. ATRIBUINDO A INCLINAÇÃO MÉDIA DA VIA PARA O BANCO DE DADOS DOS GRAFOS

A estrutura de repetição percorrerá da linha com id 1 até a linha com id máximo da tabela `vias_grafos`. Cada repetição consultará qual a declividade média de cada arco dividindo cada elemento em partes iguais de forma que cada segmento não seja maior que 50 metros. Dessa forma os campos de declividade média e declividade média absoluta serão preenchidos.

    #NOME DAS TABELAS E DOS ATRIBUTOS DA CONSULTA:
    tabela_grafos = 'vias_grafos' #NOME DE TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    abr_grafos = 'vg' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS GRAFOS DA MALHA
    atributo_dados_grafo = 'declividade_media' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE DECLIVIDADE
    atributo_dados_grafo_abs = 'declividade_media_abs' #NOME DA COLUNA (EM BRANCO) QUE SERÃO ADICIONADAS AS INFORMAÇÕES DE DECLIVIDADE ABSOLUTA
    tabela_dados = 'mde_vicosa' #NOME DE TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    abr_dados = 'mde' #ABREVIAÇÃO PARA A TABELA QUE CONTÉM O SHP DOS NOMES DAS RUAS DA MALHA
    
    #ADICIONANDO A DECLIVIDADE DAS RUAS PARA CADA ID DA TABELA DOS GRAFOS DA MALHA
    
    for id_i in range(1, id_max + 1): #OS COMANDOS SERÃO EXECUTADOS DO GRAFO DE ID=1 ATÉ O ÚLTIMO GRAFO DA TABELA QUE POSSUI ID = ID_MAX
        
        #COMPRIMENTO DO GRAFO
    
        sql = f'SELECT ST_Length(geom) FROM {tabela_grafos} WHERE id = {id_i}' #COMANDO EM SQL PARA CONSULTAR O COMPRIMENTO DO GRAFO
    
        cur.execute(sql) #EXECUTANDO O COMANDO
    
        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    
        comp_grafo = dados_consultados[0][0] #COMPRIMENTO DO GRAFO
    
        qnt_partes = math.ceil(comp_grafo/50) #QUANTIDADE DE PARTES QUE O GRAFO SERÁ DIVIDIO. CADA PONTO AO LONGO DA LINHA TERÁ DISTANCIA DE NO MÁXIMO 50m.
    
        percent = 0 # CONTADOR QUE INDICA QUAL A DISTÂNCIA (MEDIDA EM PORCENTAGEM) DA ORIGEM DA LINHA EM ANÁLISE.
    
        percent_parte = 1/qnt_partes # VARIÁVEL QUE REPRESENTA DE QUANTOS EM QUANTOS % SERÃO CALCULADAS AS DISTÂNCIAS ENTRE OS PONTOS AO LONGO DA LINHA
    
        #SELECIONANDO AS ALTITUDES AO LONGO DO GRAFO DE ACORDO COM O COMPRIMENTO DO GRAFO:
    
        lista_dados = [] #CRIANDO UMA LISTA VAZIA, NO QUAL OS DADOS CONSULTADOS SERÃO ADICIONADOS
    
        while percent <= 1: #ESTURTURA DE REPETIÇÃO PARA CRIAR E CALCULAR A DECLIVIDADE DE PONTOS AO LONGO DA LINHA EM ANÁLISE
    
            sql = f"SELECT ST_Value({abr_dados}.rast, pts.geom) as elevacao FROM (SELECT ST_LineInterpolatePoint(ST_LineMerge(geom), {percent}) as geom FROM {tabela_grafos} WHERE id = {id_i}) as pts, {tabela_dados} {abr_dados} WHERE ST_Intersects({abr_dados}.rast, pts.geom);" #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO AS COTAS DOS PONTOS AO LONGO DO GRAFO PARA O ID DO GRAFO EM ANÁLISE.
    
            cur.execute(sql) #EXECUTANDO O COMANDO
    
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS            
    
            lista_dados.append(dados_consultados[0][0])
    
            percent = percent + percent_parte # AUMENTANDO O CONTADOR PARA PASSAR PARA O PRÓXIMO PONTO AO LONGO DA LINHA
    
        #CALCULANDO A DECLIVIDADE MEDIA DO GRAFO
        decliv = 0
        for i in range(0, len(lista_dados)-1):
            decliv = decliv + (lista_dados[i+1] - lista_dados[i])/(comp_grafo/qnt_partes)
    
        decliv = round((decliv/qnt_partes)*100, 2)
    
        #ATUALIZANDO A TABELA DO SHP DOS GRAFOS COM AS INFORMAÇÕES DOS ATRIBUTOS OBTIDOS:
    
        sql = f"update {tabela_grafos} set {atributo_dados_grafo}='{decliv}' where id={id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
        sql = f"update {tabela_grafos} set {atributo_dados_grafo_abs} = '{abs(decliv)}' where id = {id_i}" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUIDO A TABELA DO SHP DOS GRAFOS A STRING ACIMA DE ACORDO COM O ID EM ANÁLISE.
        cur.execute(sql) #EXECUTANDO O COMANDO
        con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
    
        print(f'ID: {id_i} {decliv} {abs(decliv)} OK!') #COMANDO DESNECESSÁRIO. APENAS INDICA A FINALIZAÇÃO DAS REPETIÇÕES.
