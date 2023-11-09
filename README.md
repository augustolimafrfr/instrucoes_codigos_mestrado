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

**Os arquivos `Rede_Viaria_RSU_2021` e `declividade_pontos_cotados_SAAE` estão localizados na pasta `1_DADOS INICIAIS`.**

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

**Os cinco arquivos se encontram dentro do banco de dados que pode ser acessado através do dump salvo na pasta `2_DUMP_BANCO_DE_DADOS`.**

### 7. GERAÇÃO DO MDE

O MDE de Viçosa foi gerado a partir do arquivo `declividade_pontos_cotados_SAAE`. Para isso, utilizou-se a ferramenta `Interpolação IDW` do software `QGIS`. O novo arquivo foi recortado através da ferramenta `Recortar raster pela extensão...` com o objetivo de reduzir a extensão do MDE de forma que englobe apenas a área de estudo. Apesar desse passo não ser obrigatório, é interessante recortar o arquivo raster com o objetivo de reduzir o seu tamanho e facilitar os processamentos. O arquivo que contem o MDE de Viçosa foi nomeado como `mde_vicosa`.

**O MDE do município se encontra dentro do banco de dados que pode ser acessado através do dump salvo na pasta `2_DUMP_BANCO_DE_DADOS`.**

### 8. IMPORTANDO TODOS OS DADOS PARA O BANCO DE DADOS

Todos arquivos gerados foram importados para o banco de dados. O banco usado nessa dissertação possui o nome `dissertacao_artigo1` e possui todas extensões espaciais provenientes do PostGIS.

#### 8.1. IMPORTANDO OS ARQUIVOS DO FORMATO SHAPEFILE

Os arquivos agora devem ser importados para um banco de dados. O banco usado nessa dissertação possui o nome `dissertacao_v2` e possui todas extensões espaciais provenientes do PostGIS.

Os arquivos `vias_grafos`, `vias_atr_nomeruas`, `vias_atr_classpm`, `vias_atr_oneway`, `vias_atr_pavimento` e `vias_atr_largura` foram importados através da ferramenta `Gerenciador BD...` disponível no software `QGis`. O procedimento é feito selecionando de forma individual o arquivo que será importado na aba **Entrada**, em **Esquema** seleciona-se a opção _public_ e em **Tabela** atribuiu-se o nome desejado da tabela (igual ao nome do arquivo em formato shapefile). Isso foi feito para todos os arquivos.

#### 8.2. IMPORTANDO OS ARQUIVOS DO FORMATO RASTER

O MDE foi importado através do **promp de comando**. Com o promp aberto digita-se os seguintes comandos:

**PARA TRANSFORMAR O ARQUIVO MATRICIAL EM UMA TABELA SQL BINÁRIA:**

    raster2pgsql -s 31983 -I -C -M mde_vicosa.tif -F -t 100x100 public.mde_vicosa > mde_vicosa.sql

**PARA IMPORTAR O ARQUIVO NO BANCO DE DADOS:**

    psql -h localhost -U postgres -p 5432 -d dissertacao -f mde_vicosa.sql

### 9. COMPLETANDO O ARQUIVO VIAS_GRAFOS

Os atributos do arquivo `vias_grafos` estão majoritariamente em branco. Para preenchê-los utilizou-se da lingaguem Python através da IDE `Jupyter Notebook`. O procedimento necessita da instalação, através do prompt de comando, do pacote `psycopg2` (necessário para manipular o banco de dados dentro da linaguem Python). Depois disso é possível executar os comandos em lingaguem Python.

**Todos os scripts explicados estão localizados na pasta `SCRIPTS_DISSERTACAO`. Essa pasta se divide em duas sub-pastas: SCRIPTS_SQL (com os codigos executados em linguagem SQL) e SCRIPTS_PYTHON (com o ambiente virtual utilizado nesse trabalho). Para acessar os Scripts que estão separados individualmente é necessário acessar a pasta projetos que está localizada em SCRIPTS_PYTHON >> av_dissertacao >> projetos.**

#### 9.1. INSTALAÇÃO DO PACOTE psycopg2

Para instalar o pacote psycopg2 é necessário digitar o seguinte comando no **prompt de comandos**:

    pip install psycopg2
    
#### 9.2. SCRIPTS EM PYTHON:

Os comandos descritos a seguir foram todos executados no **Jupyter Notebook**.

#### 9.2.1. IMPORTANDO O PACOTE psycopg2 e math:

    import psycopg2 as pg
    import math
    
#### 9.2.2. CONECTANDO COM O BANCO DE DADOS:

Inseriu-se as configurações do banco de dados. Ele está sendo utilizado no servidor local (host = 'localhost'), o nome do banco de dados é dissertacao_v2 (database = 'dissertacao_v2'), o nome de usuário é postgres (user='postgres') e a senha do banco de dados é admin (password = 'admin').

    con = pg.connect(host='localhost', 
                    database='dissertacao_v2',
                    user='postgres', 
                    password='admin')
    
    cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL
    
    # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado DISSERTAÇÃO, que possui usuário postgres e senha admin.


#### 9.2.3. VENDO QUANTOS ARCOS A MALHA POSSUI

É necessário descobrir quantos arcos a rede possui. Para isso executa-se os seguintes comandos:

    tabela_grafos = 'vias_grafos' #TABELA COM OS GRAFOS
    sql = f'select max(id) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
    cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
    dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
    id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 9.2.4. ATRIBUINDO NOME DAS VIAS PARA O BANCO DE DADOS DOS GRAFOS

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

#### 9.2.5. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

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

#### 9.2.5.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 9.2.5.2. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

     list_comp.count(False)
     
#### 9.2.6. ATRIBUINDO O TIPO DA VIA DE ACORDO COM O PLANO DE MOBILIDADE PARA O BANCO DE DADOS DOS GRAFOS

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

#### 9.2.7. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

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
       

#### 9.2.7.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 9.2.7.2. QUANTIDADE DE FALTOS (SOMA DOS COMPRIMENTOS)

    list_comp.count(False)

#### 9.2.8. ATRIBUINDO O SENTIDO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

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

#### 9.2.9. ATRIBUINDO A PAVIMENTAÇÃO DA VIA PARA O BANCO DE DADOS DOS GRAFOS

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

#### 9.2.10. CONFERINDO A CONSISTÊNCIA DAS INFORMAÇÕES ADICIONADAS

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
       

#### 9.2.10.1. QUANTIDADE DE FALSOS (QUANTIDADE DE ITENS)

    list_len.count(False)
    
#### 9.2.10.2. QUANTIDADE DE FALTOS (SOMA DOS COMPRIMENTOS)

    list_comp.count(False)

#### 9.2.11. ATRIBUINDO A LARGURA DA VIA PARA O BANCO DE DADOS DOS GRAFOS

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


#### 9.2.12. ATRIBUINDO A INCLINAÇÃO MÉDIA DA VIA PARA O BANCO DE DADOS DOS GRAFOS

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

#### 9.2.13. ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

Após a conclusão dos processos acima, a tabela `via_grafos` foi preenchida e a conexão com o banco de dados pode ser encerrada com os seguintes comandos:

    cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO
    con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

### 10. CÁLCULO DA CONECTIVIDADE E ACESSIBILIDADE (SEM PONDERAÇÃO)

Essa parte foi dividida em duas etapas: pré-processamento e cálculos. Foi utilizados comandos em linguaguem `SQL` e linguaguem `Python` nessas etapas. Nos título dos tópicos terá informações de os comandos foram executados no pgAdmin (comando SQL) ou no Jupyter Notebook (comando Python).

#### 10.1. PRÉ-PROCESSAMENTO

#### 10.1.1. CRIAÇÃO DA REDE A SER ANALISADA `(SQL)`

Uma cópia da tabela `via_grafos` foi criada e nomeada como `rede_vicosa` com o seguinte comando:

        -- CRIANDO UMA NOVA TABELA SIMILAR AO 'vias_grafos' CHAMADA 'rede_vicosa':
        SELECT * INTO rede_vicosa FROM vias_grafos ORDER BY id;

#### 10.1.2. CRIAÇÃO DE NOVAS COLUNAS PARA A TABELA `(SQL)`

Para resolver problemas de rede com a extensão `pgRouting` é necessário a criação das colunas de nó inicial, no final, custo e custo reverso. Utiliza-se o seguinte comando:

        -- CRIANDO OS CAMPOS PARA OS VÉRTICES DE FIM E INICIO DO GRAFO:
        ALTER TABLE rede_vicosa
        ADD source INT4,
        ADD target INT4,
        ADD cost REAL,
        ADD reverse_cost REAL;

#### 10.1.3. RENOMEANDO O CAMPO 'geom' PARA 'the_geom' `(SQL)`

        -- RENOMEANDO O CAMPO 'geom' PARA 'the_geom':
        ALTER TABLE rede_vicosa
        RENAME COLUMN geom TO the_geom;

#### 10.1.4. REDEFININDO OS DADOS DE DIREÇÃO DA VIA PARA 'YES' OU 'NO' `(SQL)`

        -- REDEFININDO OS DADOS DE DIREÇÃO DA VIA PARA 'YES' OU 'NO':
        -- VIAS COM MÃO ÚNICA:
        UPDATE rede_vicosa SET oneway = 'YES' WHERE oneway = 'TF';
        
        -- VIAS COM MÃO DUPLA:
        UPDATE rede_vicosa SET oneway = 'NO' WHERE oneway = 'B';

 #### 10.1.5. DEFININDO OS CUSTOS DOS ARCOS `(SQL)`

        -- DEFININDO CUSTOS 1 PARA TODOS OS VÉRTICES:
        UPDATE rede_vicosa SET cost = 1, reverse_cost = 1;
        
        -- DEFININDO O CUSTO REVERSOS = -1 PARA OS GRAFOS COM DIREÇÃO ÚNICA:
        UPDATE rede_vicosa SET reverse_cost = '-1' WHERE oneway = 'YES';

#### 10.1.6. CRIANDO A TOPOLOGIA DA REDE `(SQL)`

    SELECT pgr_createTopology('rede_vicosa', 1);

As colunas source e target serão preenchidas com id's de vérticies iniciais e finais. Uma nova tabela surgirá com a geometria dos vértices gerados. 

#### 10.1.7. CRIANDO VÉRTICES NO MEIO DAS LINHAS `(SQL)`

Para o cálculo da acessibilidade é necessário computar o custo entre arcos, porém, a extensão só permite o cálculo entre nós. Para contornar esse problema, criou-se nós no meio de cada linha. O custo para atravessar esses nós centrais (com o custo de cada metade de arco igual a 0,5) é igual o custo entre arcos.

A tabela com pontos intermediários chama-se `mid_points` e para criá-la usou o seguinte comando:

        -- CRIANDO OS VÉRTICES NO MEIO DE CADA GRAFO:
        CREATE TABLE mid_points AS
        SELECT id, ST_LineInterpolatePoint(ST_LineMerge(the_geom), 0.5) as geom
        FROM rede_vicosa;
        
        ALTER TABLE mid_points
        ADD CONSTRAINT mid_points_pk PRIMARY KEY (id);
        
        CREATE INDEX sidx_mid_points
         ON mid_points
         USING GIST (geom);

#### 10.1.8. VERIFICAR SE AS COLUNAS SOURCE E TARGET ESTÃO CORRETAS `(SQL)`

Foi necessário verificar no `QGIS` se as colunas source e target estão preenchidas corretamente com os valores de id's dos vértices para as linhas com sentido de via unidirecional. Foi possível constatar que as linhas com id 1, 60, 65, 76 e 104 estavam com o sentido invertido e isso foi consertado com os seguintes comandos:

    -- CONFERIR SE OS VÉRTICES SOURCE E TARGET ESTÃO DE ACORDO COM A REALIDADE PARA OS GRAFOS QUE POSSUEM APENAS UMA MÃO.
    -- DE FORMA MANUAL, DEVE-SE OBSERVAR AQUELES GRAFOS QUE NECESSIATARAM DE ALTERAR O CAMPO 'source' e 'target' DURANTE A EDIÇÃO DA CAMADA 'rede_vicosa'
    
    -- id do grafo: 1
    UPDATE rede_vicosa
    SET source = '41', target = '11'
    WHERE id = 1;
    
    -- id do grafo: 60
    UPDATE rede_vicosa
    SET source = '36', target = '47'
    WHERE id = 60;
    
    -- id do grafo: 65
    UPDATE rede_vicosa
    SET source = '77', target = '48'
    WHERE id = 65;
    
    -- id do grafo: 76
    UPDATE rede_vicosa
    SET source = '88', target = '78'
    WHERE id = 76;
    
    -- id do grafo: 104
    UPDATE rede_vicosa
    SET source = '73', target = '62'
    WHERE id = 104;

#### 10.1.9. CRIANDO A REDE COM PONTOS INTERMEDIÁRIOS ATRAVÉS DO QGIS
 
Para criar a rede com pontos intermediários é necessário usar o software `QGIS` através da ferramenta `Connect nodes to lines` presente no complemento `Networks`. Criou-se uma cópia da camada `rede_vicosa` renomeando-a para `rede_vicosa_mp_prov` e a ferramenta foi utilizada nessa nova camada e o resultado foi importado para dentro do banco de dados utilizando o Gerenciador BD do `QGIS`. Durante a importação o id dessa rede foi nomeado como id0.

#### 10.1.10. REMOVENTO ELEMENTOS INUTEIS DA `rede_vicosa_mp_prov` `(SQL)`
 
A criação dessa nova rede não resultou em um arquivo pronto para ser trabalhado e novos processamento foram realizados para "limpar" a tabela. A tabela limpa foi nomeada como `rede_vicosa_mp` e os comandos foram os seguintes:

        -- REMOVENDO DA CAMADA rede_vicosa_mp AQUELAS GEOMETRIAS NÃO UTEIS PARA A ATUAL ANÁLISE        
        CREATE TABLE rede_vicosa_mp AS
        SELECT rvmpp.id, rvmpp.comp_via, rvmpp.nome_rua, rvmpp.comp_nome_, rvmpp.class_pm, rvmpp.comp_class, rvmpp.oneway, rvmpp.pavimento, rvmpp.comp_pavim, rvmpp.largura_me, rvmpp.declividad, rvmpp.declivid_1,  rvmpp.source, rvmpp.target, rvmpp.cost, rvmpp.reverse_co, ST_Difference(rvmpp.geom, ptos.geom) as geom 
        FROM (SELECT rvmpp.*
        FROM rede_vicosa_mp_prov rvmpp, mid_points mp
        WHERE ST_Contains(mp.geom, rvmpp.geom)) as ptos, rede_vicosa_mp_prov rvmpp
        WHERE ST_Intersects(rvmpp.geom, ptos.geom);
        
        DELETE FROM rede_vicosa_mp
        WHERE ST_LENGTH(geom) = 0;
        
        ALTER TABLE rede_vicosa_mp
        ADD COLUMN id0 SERIAL PRIMARY KEY;
        
        --DELETANDO A TABELA rede_vicosa_mp_prov, JÁ QUE A DEFINITIVA FOI CRIADA
        DROP TABLE rede_vicosa_mp_prov;
        
        -- ALTERANDO A TABELA REDE_VICOSA_MP:
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN id TO id_grafo;

#### 10.1.11. RENOMEANDO O id DA TABELA `mid_points` `(SQL)`

A tabela mid_points apresenta o id de cada ponto igual ao id de sua respectiva linha da tabela `rede_vicosa`. Para diferenciar os pontos intermediários dos pontos finais e iniciais de cada linha original da `rede_vicosa`, escolheu-se somar 1000 ao valor do id do ponto intermediário. Dessa forma fica fácil de saber que o ponto de id 1050 da tabela `rede_vicosa_mp` corresponde ao ponto intermediário da linha 50, enquanto o ponto com id 50 é algum vértice inicial e/ou final de alguma linha. O comando utilizado foi:

        --ALTERANDO A TABELA 'mid_points' PARA INCLUIR UMA COLUNA CHAMADA ID_GRAFO E ALTERAR O ID PARA 1000 + id DO GRAFO.
        ALTER TABLE mid_points
        ADD id_grafo INT4;
        
        UPDATE mid_points
        SET id_grafo = id;
        
        UPDATE mid_points
        SET id = 1000 + id;
        
        SELECT * FROM mid_points;

#### 10.1.12. CRIANDO A TABELA `rede_vicosa_mp_vertices` COM TODOS OS VÉRTICES DA `rede_vicosa_mp` (VÉRTICES DE INÍCIO E FIM DA `rede_vicosa` E VÉRTICES DA TABELA `mid_points` `(SQL)`

        --CRIANDO A TABELA rede_vicosa_mp_vertices QUE POSSUIRÁ TODOS OS VERTICES DA REDE PARA CALCULO DE ACESSIBILIDADE:
        SELECT *
        INTO rede_vicosa_mp_vertices
        FROM mid_points;
        
        ALTER TABLE rede_vicosa_mp_vertices
        ADD CONSTRAINT rede_vicosa_mp_vertices_pk PRIMARY KEY (id);
        
        INSERT INTO rede_vicosa_mp_vertices (id, geom)
        (SELECT id, the_geom as geom FROM rede_vicosa_vertices_pgr);
        
        SELECT * FROM rede_vicosa_mp_vertices;

#### 10.1.13. COMPLETANDO A TABELA `rede_vicosa_mp` COM OS VÉRTICES INICIAIS E FINAIS DE CADA LINHA `(PYTHON)`
 
Para completar as colunas source e target da tabela `rede_vicosa_mp` utilizou-se um código em linguagem Python. O código realiza uma estrutura de repetição que percorre da linha 1 até a linha de id máximo da tabela e analisa qual elemento da tabela `rede_vicosa_mp_vertices` está contido no inicio e fim da linha em análise. Em seguida atualiza as colunas source e target com os valores encontrados. Segue o código:

#### 10.1.13.1. IMPORTANDO O PACOTE psycopg2 QUE CONECTA O PYTHON COM O POSTGRE-SQL E O PACOTE pandas PARA ORGANIZAR AS TABELAS DE ACESSIBILIDADE `(PYTHON)`

    import psycopg2 as pg

#### 10.1.13.2. CONECTANDO AO BANCO DE DADOS `(PYTHON)`

        con = pg.connect(host='localhost', 
                        database='dissertacao_v2',
                        user='postgres', 
                        password='admin')
        
        cur = con.cursor() #CRIANDO UMA INSTÂNCIA PARA EXECUTAR COMANDOS EM SQL
        
        # OBS: O servidor hospedado na máquina local será conectado no banco de dados nomeado rede_exemplo, que possui usuário postgres e senha admin.

#### 10.1.13.3. VENDO QUANTOS ARCOS A NOVA MALHA POSSUI `(PYTHON)`

        tabela_grafos = 'rede_vicosa_mp' #TABELA COM A REDE
        tabela_vertices = 'rede_vicosa_mp_vertices'
        sql = f'select max(id0) from {tabela_grafos}' #COMANDO EM SQL A SER EXECUTADO
        cur.execute(sql) #EXECUTANDO O COMANDO CRIADO
        dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
        id_max = dados_consultados[0][0] #ID MÁXIMO DA MALHA

#### 10.1.13.4. DEFININDO OS VÉRTICES DE INICIO E FIM `(PYTHON)`

        for id_i in range (1, id_max + 1):
            #CONSULTANDO O VÉRTICE INICIAL DO GRAFO i:
        
            sql = f'SELECT rvmp.id, rvmp.geom FROM (SELECT ST_LineInterpolatePoints(ST_LineMerge(geom), 0) as geom FROM {tabela_grafos} WHERE id0 = {id_i}) as pto, {tabela_vertices} rvmp WHERE ST_Intersects(pto.geom, rvmp.geom)' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE INICIAL DO GRAFO i
        
            cur.execute(sql) #EXECUTANDO O COMANDO
        
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
            
            id_ini = dados_consultados[0][0]
            
            #CONSULTANDO O VÉRTICE FINAL DO GRAFO i
        
            sql = f'SELECT rvmp.id FROM (SELECT ST_LineInterpolatePoints(ST_LineMerge(geom), 1) as geom FROM {tabela_grafos} WHERE id0 = {id_i}) as pto, {tabela_vertices} rvmp WHERE ST_Intersects(pto.geom, rvmp.geom)' #COMANDO EM SQL A SER EXECUTADO. SERÁ SELECIONADO O VÉRTICE FINAL DO GRAFO i
        
            cur.execute(sql) #EXECUTANDO O COMANDO
        
            dados_consultados = cur.fetchall() #RETORNANDO OS DADOS
            
            id_fim = dados_consultados[0][0]
        
            #ATUALIZANDO A TABELA DOS GRAFOS DA REDE:
        
            sql = f"update {tabela_grafos} set source ='{id_ini}', target = '{id_fim}' where id0 = {id_i};" #COMANDO EM SQL A SER EXECUTADO. SERÁ ATRIBUITO A TABELA DOS GRAFOS A SOURCE E TARGET ENCONTRADA.
        
            cur.execute(sql) #EXECUTANDO O COMANDO
        
            con.commit() #FINALIZANDO A EXECUÇÃO DO COMANDO
            
            print(f'ID: {id_i} -> v_ini = {id_ini} e v_fim = {id_fim}')
        
        cur.close() #ENCERRANDO A INSTÂNCIA CRIADA PARA A EXECUÇÃO DO COMANDO
        
        con.close() #ENCERRANDO A CONEXÃO COM O BANCO DE DADOS

#### 10.1.13.5. ENCERRANDO A CONEXÃO `(PYTHON)`

        cur.close()
        con.close()

#### 10.1.14. VERIFICANDO VÉRTICES INICIAIS E FINAIS DAS LINHAS COM SENTIDO UNIDIRECIONAL `(SQL)`

Novamente é necessário verificar no `QGIS` se os campos source e target das linhas com sentido unidirecional estão corretos. Mas mesmas linhas com problemas descritas no `tópico 12.8` apresentarão problemas. São elas: id 1,2, 37,38, 51, 52, 63 e 64. Para corrigir utilizou-se o seguinte comando dentro do pgAdmin:

        -- id do grafo: id_grafo = 1 e id0 = 158, 157
        UPDATE rede_vicosa_mp
        SET source = '41', target = '1001'
        WHERE id0 = 158;
        
        ------
        UPDATE rede_vicosa_mp
        SET source = '1001', target = '11'
        WHERE id0 = 157;
        
        -- id do grafo: id_grafo = 60 e id0 = 196, 195
        UPDATE rede_vicosa_mp
        SET source = '36', target = '1060'
        WHERE id0 = 196;
        
        ------
        UPDATE rede_vicosa_mp
        SET source = '1060', target = '47'
        WHERE id0 = 218;
        
        -- id do grafo: id_grafo = 65 e id0 = 202 e 201
        UPDATE rede_vicosa_mp
        SET source = '77', target = '1065'
        WHERE id0 = 202;
        
        -------
        UPDATE rede_vicosa_mp
        SET source = '1065', target = '48'
        WHERE id0 = 201;
        
        -- id do grafo: id_grafo = 76 e id0 = 230 e 229
        UPDATE rede_vicosa_mp
        SET source = '88', target = '1076'
        WHERE id0 = 230;
        
        -----
        UPDATE rede_vicosa_mp
        SET source = '1076', target = '78'
        WHERE id0 = 229;
        
        -- id do grafo: id_grafo = 104 e id0 = 246 e 245
        UPDATE rede_vicosa_mp
        SET source = '73', target = '1104'
        WHERE id0 = 246;
        
        -----
        UPDATE rede_vicosa_mp
        SET source = '1104', target = '62'
        WHERE id0 = 245;

#### 10.1.15. RENOEMANDO ALGUMAS COLUNAS DA REDE E RECRIANDO A TOPOLOGIA DA REDE:

        -- RENOEMANDO ALGUMAS COLUNAS DA REDE E RECRIANDO A TOPOLOGIA DA REDE:
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN geom TO the_geom;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN id0 TO id;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN comp_nome_ TO comp_nome_rua;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN comp_class TO comp_class_pm;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN comp_pavim TO comp_pavimento;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN largura_me TO largura_media;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN declividad TO declividade_media;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN declivid_1 TO declividade_media_abs;
        
        ALTER TABLE rede_vicosa_mp
        RENAME COLUMN reverse_co TO reverse_cost;
        
        SELECT pgr_createTopology('rede_vicosa_mp', 1);

 Após esse processo a rede está pronta para o cálculo da acessibilidade.
 
 **Os arquivos `rede_vicosa`, `rede_vicosa_vertices_pgr`, `rede_vicosa_mp` e `rede_vicosa_mp_vertices` estão localizados dentro do banco de dados que pode ser acessado através do dump salvo na pasta `2_DUMP_BANCO_DE_DADOS`.**


































































































































