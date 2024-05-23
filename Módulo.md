# Redes-complexas
Código em python que coleta medidas de conectividade, centralidade e transitividade de redes complexas.
Primeiramente, temos as bibliotecas que foram utilizadas para esse módulo. São ao todo 3 bibliotecas utilizadas.

    import networkx as nx
    import numpy as np
    import matplotlib.pyplot as plt

Temos, primeiramente, uma função para calcular o grau médio das redes. Ele possui uma otimização muito boa, sendo muito rápido.

    def cal_graumedio(B):
        N = len(B.nodes)
        Ni = np.zeros(N)
        k = 0
        soma = 0
        for node in B.nodes():
            k = B.degree[node]
            node = int(node)
            Ni[node] = k
            soma = soma + k

        grau_medio = (1/N) * soma
        return(grau_medio)

Em sequência, temos um cálculo feito para a densidade de probabilidade, utilizado para fazermos o gráfico de probabilidade por grau. Isso será gerado adiante, mas essa função é gerada separadamente para facilitar indentificação de erros. Assim, podemos conferir a lei de potência, propriedade de redes sem escala.

    def graf_probabxgrau(B):
        N = len(B.nodes)
        Ni = np.zeros(N)
        k = 0
        kmáx = 0

        for node in B.nodes():
            k = B.degree[node]
            node = int(node)
            Ni[node] = k
            if Ni[node] > kmáx:
                kmáx = Ni[node]

        kmáx = int(kmáx)

        NK = np.zeros(kmáx+1)
        PK = np.zeros(kmáx+1)
        Nk = 0
        k = 0

        for k in range(0,kmáx+1):
            for j in range(0,N):
                if Ni[j] == k:
                    Nk = Nk + 1
            NK[k] = Nk
            PK[k] = (Nk/N)
            Nk = 0

        soma = 0
        for k in range(0,kmáx+1):
            soma = soma + PK[k]

        K = list(set(Ni))

        for i in range(0,len(K)):
            K[i] = int(K[i])

        for t in range(len(PK)-1,-1,-1):
            if t < len(PK):
                if PK[t] == 0:
                    PK = np.delete(PK, t)
            else:
                break
        return(K, PK)

Assim, geramos o seguinte gráfico, que pode ser analisado após para compreender nossa rede.

    def draw_probabxgrau(K,PK):
        plt.loglog()
        plt.scatter(K, PK, marker = "o")
        plt.xlabel('log(K)')
        plt.ylabel('log(P(k))')
        plt.title('Distribuição de grau')
        plt.grid(True)
    plt.show()

Para calcular o clustering local, foi modulado um modelo arcaico, uma função mais lenta, que viria a ser melhorada a seguir com uma arquitetura de código mais robusta. Essa função nos dá o clustering da rede partindo do cálculo local, assim como a função mais eficiente.

    def cal_clustering_local(B):
        N = len(B.nodes)
        e = 0
        Cl = 0

        for node in B.nodes():
            ki = B.degree(node)
            V = list(B.neighbors(node))
            for r in range(0,len(V)):
                VDV = list(B.neighbors(V[r]))
                for i in range(0,len(V)):
                    for j in range(0,len(VDV)):
                        if V[i] == VDV[j]:
                            e = e + 1
            if ki != 1:
                Ci = (2*e)/(ki*(ki-1))
                Cl = Cl + (Ci/N)
            e = 0
        return(Cl, Ci)

Agora, sim, utilizando o método aprimorado, com uma arquitetura de código mais robusta, esse código calcula mais rapidademente o clustering local. Utilizando já da função do mólulo networkx. Logo, temos uma nova função, mais eficiente que a anterior.
        
    def cal_clustering_local_networkx(B):
    
        CL = nx.clustering(B)
        Ci = np.zeros(len(CL))
        Cluster = np.array(list(CL.items()))
        Clus = np.zeros(2)
        soma = 0
        for i in range(0,len(B.nodes)):
            Clus = Cluster[i]
            Clus = [ float(x.strip("[]").replace(",", ".")) for x in Clus ]
            for j in range(0,2):
                soma = soma + Clus[1]
                Ci[i] = Clus[1] 
        Cl = soma/len(B.nodes)
        print(Ci)
        return(Cl, Ci)

O próximo passo seria gerar um gráfico do clustering individual pelo grau. Ainda está incompleto, mas já estou idealizando maneiras de fazer isso.
        
    def cal_clustering_for_degree(B, K, Ci):
        
        for node in B.nodes():
            if K[node] == B.degree(node):
                dado = dado
                
Agora, partindo para centralidade, temos um cálculo mais arcaico da centralidade pelo autovetor. Digo isso porque criei um modelo mais rápido que o descrito abaixo, que será visto mais à frente.
        
    def cal_centralidade_eingevector(B):
        N = len(B.nodes)

        Adj = nx.adjacency_matrix(B)
        Adjunt = Adj.toarray()

        X = np.zeros(N)
        for i in range(0,N):
            X[i] = 1

        XT = np.transpose(X)

        for i in range(0,5):
            XT = np.dot(XT,Adjunt)

        soma = 0
        for i in range(0,len(XT)):
            soma = soma + XT[i]

        for i in range(0,len(XT)):
            XT[i] = XT[i]/soma

        soma1 = 0
        for i in range(0,len(XT)):
            soma1 = soma1 + XT[i]
        máximo = max(XT)
        posição = np.where(XT == máximo)

        m = len(XT)

        XT_b = [0] * m 
        XT_c = [0] * m 
        XT_g = [0] * m 
        XT_y = [0] * m 
        XT_r = [0] * m 

        for i in range(0,len(XT)):
            if XT[i] < 0.2*máximo:
                XT_b[i] = XT[i]
            elif XT[i] < 0.4*máximo:
                XT_c[i] = XT[i]
            elif XT[i] < 0.6*máximo:
                XT_g[i] = XT[i]
            elif XT[i] < 0.8*máximo:
                XT_y[i] = XT[i]
            else:
                XT_r[i] = XT[i]

        posiçõesb = np.zeros(m)
        posiçõesc = np.zeros(m)
        posiçõesg = np.zeros(m)
        posiçõesy = np.zeros(m)
        posiçõesr = np.zeros(m)

        for i in range(0,len(XT)):
            if XT_b[i] != 0:
                posiçõesb[i] = XT_b.index(XT_b[i])
            XT_b[i] = 0
            if XT_c[i] != 0:
                posiçõesc[i] = XT_c.index(XT_c[i])
            XT_c[i] = 0
            if XT_g[i] != 0:
                posiçõesg[i] = XT_g.index(XT_g[i])
            XT_g[i] = 0
            if XT_y[i] != 0:
                posiçõesy[i] = XT_y.index(XT_y[i])
            XT_y[i] = 0
            if XT_r[i] != 0:
                posiçõesr[i] = XT_r.index(XT_r[i])
            XT_r[i] = 0

        pos_b = posiçõesb.tolist()
        pos_c = posiçõesc.tolist()
        pos_g = posiçõesg.tolist()
        pos_y = posiçõesy.tolist()
        pos_r = posiçõesr.tolist()

        return(pos_b, pos_c, pos_g, pos_y, pos_r)

Como já dito, o código anterior era arcaico e lento. Por isso, foi aplicada uma nova função, que é mais rápida que a anterior.

    def cal_centralidade_eingevector_networkx(B):
        Ct = nx.eigenvector_centrality(B)

        tam = len(B.nodes)

        Ct_b = [0] * tam
        Ct_c = [0] * tam
        Ct_g = [0] * tam
        Ct_y = [0] * tam
        Ct_r = [0] * tam

        value = np.zeros(len(B.nodes))

        Central = np.array(list(Ct.items()))

        for i in range(0,len(B.nodes)):
            Ct = Central[i]
            value[i] = Ct[1]

        máximo = max(value)
        máximo = float(máximo)

        for i in range(0,len(B.nodes)):
            Ct = Central[i]
            Ct = [ float(x.strip("[]").replace(",", ".")) for x in Ct ]
            if Ct[1] < 0.2*máximo:
                Ct_b[i] = Ct[0]
            elif Ct[1] < 0.4*máximo:
                Ct_c[i] = Ct[0]
            elif Ct[1] < 0.6*máximo:
                Ct_g[i] = Ct[0]
            elif Ct[1] < 0.8*máximo:
                Ct_y[i] = Ct[0]
            else:
                Ct_r[i] = Ct[0]
        return(Ct_b, Ct_c, Ct_g, Ct_y, Ct_r)

Portanto, agora que temos os valores de centralidade, podemos encontrar os pontos mais centrais e fazermos um mapeamento no grafo.

    def draw_centralidade(B, pos_b, pos_c, pos_g, pos_y, pos_r):
        for t in range(len(B.nodes)-1,-1,-1):
            if t < len(B.nodes):
                if pos_b[t] == 0:
                    pos_b = np.delete(pos_b, t)
                if pos_c[t] == 0:
                    pos_c = np.delete(pos_c, t)
                if pos_g[t] == 0:
                    pos_g = np.delete(pos_g, t)
                if pos_y[t] == 0:
                    pos_y = np.delete(pos_y, t)
                if pos_r[t] == 0:
                    pos_r = np.delete(pos_r, t)
            else:
                break
    
        pos_b = pos_b.tolist()
        pos_c = pos_c.tolist()
        pos_g = pos_g.tolist()
        pos_y = pos_y.tolist()
        pos_r = pos_r.tolist()
    
        for i in range(0,len(pos_b)):
            pos_b[i] = int(pos_b[i])
        for i in range(0,len(pos_c)):    
            pos_c[i] = int(pos_c[i])
        for i in range(0,len(pos_g)): 
            pos_g[i] = int(pos_g[i])
        for i in range(0,len(pos_y)): 
            pos_y[i] = int(pos_y[i])
        for i in range(0,len(pos_r)): 
            pos_r[i] = int(pos_r[i])
            
        no = np.zeros(len(B.nodes))
    
        for node in B.nodes():
            Vzn = list(B.neighbors(node))
            node = int(node)
            no[node] = node
        
        arquivo = open("grafo_b.txt", "w")
    
        for k in range(0,len(B.nodes)):
            for i in range(0,len(pos_b)):
                if pos_b[i] == no[k]:
                    pos_b[i] = str(pos_b[i])
                    for j in range(0,len(Vzn)):    
                        arquivo.write("%s %s \n" %(pos_b[i],Vzn[j]))    
        arquivo.close()
    
        arquivo = open("grafo_c.txt", "w")

        for k in range(0,len(B.nodes)):
            for i in range(0,len(pos_c)):
                if pos_c[i] == no[k]:
                    pos_c[i] = str(pos_c[i])
                    for j in range(0,len(Vzn)):    
                        arquivo.write("%s %s \n" %(pos_c[i],Vzn[j]))
        arquivo.close()
    
        arquivo = open("grafo_g.txt", "w")
    
        for k in range(0,len(B.nodes)):
            for i in range(0,len(pos_g)):
                if pos_g[i] == no[k]:
                    pos_g[i] = str(pos_g[i])
                    for j in range(0,len(Vzn)):    
                        arquivo.write("%s %s \n" %(pos_g[i],Vzn[j]))
        arquivo.close()
    
        arquivo = open("grafo_y.txt", "w")
    
        for k in range(0,len(B.nodes)):
            for i in range(0,len(pos_y)):
                if pos_y[i] == no[k]:
                    pos_y[i] = str(pos_y[i])
                    for j in range(0,len(Vzn)):    
                        arquivo.write("%s %s \n" %(pos_y[i],Vzn[j]))
        arquivo.close()
    
        arquivo = open("grafo_r.txt", "w")
    
        for k in range(0,len(B.nodes)):
            for i in range(0,len(pos_r)):
                if pos_r[i] == no[k]:
                    pos_r[i] = str(pos_r[i])
                    for j in range(0,len(Vzn)):    
                        arquivo.write("%s %s \n" %(pos_r[i],Vzn[j]))
        arquivo.close()
    
        blue = nx.read_edgelist("grafo_b.txt")
        cian = nx.read_edgelist("grafo_c.txt")
        green = nx.read_edgelist("grafo_g.txt")
        yellow = nx.read_edgelist("grafo_y.txt")
        red = nx.read_edgelist("grafo_r.txt")
        sub_b = B.subgraph(blue)
        sub_c = B.subgraph(cian)
        sub_g = B.subgraph(green)
        sub_y = B.subgraph(yellow)
        sub_r = B.subgraph(red)
        pos = nx.spring_layout(B)
        plt.figure(figsize = (10,6))
        nx.draw_networkx(B, with_labels = False, font_color = "red", node_size = 20, pos = pos)
        nx.draw_networkx(sub_b, with_labels = False, node_size = 40, pos = pos, node_color = "#1770F1")
        nx.draw_networkx(sub_c, with_labels = False, node_size = 40, pos = pos, node_color = "#38CFFF")
        nx.draw_networkx(sub_g, with_labels = False, node_size = 40, pos = pos, node_color = "#35F53B")
        nx.draw_networkx(sub_y, with_labels = False, node_size = 40, pos = pos, node_color = "#DEF200")
        nx.draw_networkx(sub_r, with_labels = False, node_size = 40, pos = pos, node_color = "#FF5900")
        plt.show()

Também podemos plotar o grafo que analisamos, pelo seguinte código:

    def draw_network(B):
    
        pos = nx.spring_layout(B)
        nx.draw(B, with_labels = False, font_color = "red", node_size = 20, pos = pos)
        plt.show()
