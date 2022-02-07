# Exame 2019 - Universidade do Minho
## Indique quais os grafos que conhece que não exibem ciclos e identifique o que lhe parecer mais apropriado para broadcast de mensagens
Grafos que possuem ciclos permitem multi-path routing, pelo que não são recomendados.
Os grafos do tipo *Star* e do tipo *Tree* não possuem ciclos.
Dentro dos grafos das *Trees* é possível identificar vários tipos de árvores: 
- *Spanning Tree*, contendo *Async Tree* e *Sync BFS Tree*.
Através deste tipo de árvores, é possível identificar que existe pouca complexidade na troca de mensagens sendo estas passadas dos nós pais para os nós filhos. No entanto, é bastante frágil relativamente a falhas.

Tendo em conta que este tipo de árvores não é muito escalável para a troca de mensagens, surgiu um novo tipo de árvores:
- *Epidemic Broadcast Trees*, using *Gossip Strategies* to broadcast messages.
O *PlumTree Protocol* é protocolo utilizado para implementar *Epidemic Broadcast Trees*. Utilizando estratégias de *Gossip* para ser escalável a sistemas distribuidos em larga escala. Além disso a recuperação caso a árvore tenha uma falha é rápida e a redundancia que pode ser originada graças a isso (por causa de ciclos) é rapidamente removida pelo algoritmo.

## Apresente uma estratégia para a determinação do diâmetro num grafo que sabemos possuir uma única connected component.
To find the diameter of a graph, first **find the shortest path between each pair of vertices**. The greatest length of any of these paths is the diameter of the graph.

## Explique de que modo os bloom filters permitiram melhorar a escabilidade da rede Gnutella
A rede Gnutella originalmente utilizava mensagens de *PING* e *PONG* para descobrir novos nós. *PINGs* are flooded and *PONGs* are answered by along reverse paths. Existiam também outros comandos como *QUERY* e *QUERY RESPONSE* que permitiam a pesquisa de informação e *PUSH* e *GET* para iniciar tranferência de ficheiros entre peers.
Esta implementação não escalava, uma vez que os comandos *PING* e *PONG* eram os dominantes na rede inteira.

Foi então criada uma versão melhorada da mesma rede. Foram criadas *Super Peers*, que são peers que estão (quase) sempre online, com alta largura de rede e preferidos para conexões com outros peers. 

Os Peers começaram também a enviar o conteúdo que possuem para os *Super Peers*. Os conteúdos são enviados através de uma hash utilizando *Bloom Filters*. O *Bloom Filter* é uma estrutura de dados probabilistica que indica a probabilidade de um determinado elemento estar presente ou não num determinado set.
Isto permitiu melhorar a escabilidade da rede uma vez que agora os *Super Peers* apenas contactam os peers que possuem uma elevada probabilidade de ter o elemento invés de pesquisar por todos.

Source -> https://people.eecs.berkeley.edu/~kubitron/courses/cs294-4-F03/projects/poon_hess.pdf
## Indique qual a relevância da latência da rede na sincronização de relógios (wall clocks) num sistema distribuído
Para sincronizar os relógios numa rede distribuido é necessário utilizar algo como o *Network Time Protocol*. Para isto, é necessário fazer um request do tempo atual a um servidor externo.

Como sabemos, as mensagens não são instantaneas e possuem latência. Assumindo que:
- A mensagem demora *t_transmission* para ser transmitida através do computador da pessoa até ao servidor externo que possui a hora correta
- O tempo recebido pela mensagem é *t_receive*

Podemos então assumir, que o tempo que o tempo atual quando a mensagem chegar ao computador da pessoa é:
*t_receive* + *t_transmission*


## Explique que tipo de problemas podem advir da utilização de relógios (wall clocks) na implementação multi-master de um registo distribuído
*Multi-master* (ou *active-active*) é um método de replicação de dados que permite aos dados serem armazenados num grupo de computadores e atualizados por qualquer elemento nesse mesmo grupo. 

Este tipo de estrutura permite ao cliente ter uma replica proxima a ele, não precisando de coordenar com outros clientes para efetuar alterações. Isto faz com que latência de acesso seja baixa, mas em contrapartida surge *divergência*. Este tipo de sistema é por exemplo utilizado no git (branchs podem ter conflitos de merge).

Deste modo, não conseguimos ter uma ordem total, sendo apenas possivel obter uma ordem parcial. Ao termos uma ordem parcial existem eventos que são concurrentes e não podemos afirmar **que um evento ocorreu antes do outro mesmo que tenha acontecido previamente utilizando uma ordem temporal**.

![[Pasted image 20220205223231.png]]

Por exemplo, na imagem, apesar de *s* ter ocorrido antes de *q* tendo em conta o tempo são eventos concorrentes.

Logo não se pode concluir que *q* < *s* ou *s* < *q* 
Para evitar estes problemas, surgiu então o *CRDT*  (Conflit-free replicated data types)