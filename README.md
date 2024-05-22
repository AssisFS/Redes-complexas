# Redes-complexas
Código em python que coleta medidas de conectividade, centralidade e transitividade de redes complexas.

import networkx as nx
import numpy as np
import matplotlib.pyplot as plt

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

def cal_clustering_global(B):
    for edge in B.edges():
        print(edge)
        
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
        
def cal_clustering_for_degree(B, K, Ci):
        
    for node in B.nodes():
        if K[node] == B.degree(node):
            dado = dado
        
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

    for t in range(len(XT)-1,-1,-1):
        if t < len(XT):
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

    return(pos_b, pos_c, pos_g, pos_y, pos_r)

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

def draw_clusteringxgrau(K,Ci):
    plt.scatter(K, Ci, marker = "o")
    plt.xlabel('K')
    plt.ylabel('CC')
    plt.title('Coeficiente de agrupamento por grau')
    plt.grid(True)
    plt.show()

def draw_centralidade(B, pos_b, pos_c, pos_g, pos_y, pos_r):
    pos = nx.spring_layout(B)
    subset_b = B.subgraph(pos_b) 
    subset_c = B.subgraph(pos_c)
    subset_g = B.subgraph(pos_g)
    subset_y = B.subgraph(pos_y)
    subset_r = B.subgraph(pos_r)
    print(subset_b,subset_c,subset_g,subset_y,subset_r)
    plt.figure(figsize = (100,60))
    nx.draw_networkx(subset_b, with_labels = True, node_size = 100, pos = pos, node_color = "#1770F1")
    nx.draw_networkx(subset_c, with_labels = True, node_size = 100, pos = pos, node_color = "#38CFFF")
    nx.draw_networkx(subset_g, with_labels = True, node_size = 100, pos = pos, node_color = "#35F53B")
    nx.draw_networkx(subset_y, with_labels = True, node_size = 100, pos = pos, node_color = "#DEF200")
    nx.draw_networkx(subset_r, with_labels = True, node_size = 100, pos = pos, node_color = "#FF5900")
    plt.show()

def draw_network(B):
    
    pos = nx.spring_layout(B)
    nx.draw(B, with_labels = False, font_color = "red", node_size = 20, pos = pos)
    plt.show()

def draw_probabxgrau(K,PK):
    plt.loglog()
    plt.scatter(K, PK, marker = "o")
    plt.xlabel('log(K)')
    plt.ylabel('log(P(k))')
    plt.title('Distribuição de grau')
    plt.grid(True)
    plt.show()
