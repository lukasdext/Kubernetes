# Kubernetes
Manter os aplicativos em containers em funcionamento pode ser complexo, pois eles geralmente envolvem muitos containers implantados em computadores diferentes. O Kubernetes fornece uma maneira de agendar e implantar esses containers, além de dimensioná-los para o estado desejado e gerenciar seus ciclos de vida. Use o Kubernetes para implementar seus aplicativos baseados em containers de maneira portátil, escalonável e extensível.
# Introdução
Kubernetes é frequentemente descrito como uma plataforma de orquestração de containers. Para entender exatamente o que isso significa, ajuda a revisitar o propósito dos containers, o que está faltando e como Kubernetes preenche essa lacuna.
Os containers fornecem um mecanismo leve para isolar o ambiente de uma aplicação. Para um determinado aplicativo, podemos especificar a configuração do sistema e bibliotecas que queremos instaladas sem se preocupar em criar conflitos com outros aplicativos que possam estar sendo executados na mesma máquina física. Encapsulamos cada aplicativo como uma imagem de container que pode ser executada de forma confiável em qualquer máquina* (desde que tenha a capacidade de executar imagens de container), fornecendo-nos a portabilidade para permitir transições suaves do desenvolvimento para a deployment. Além disso, como cada aplicativo é independente sem a preocupação com conflitos de ambiente, é mais fácil colocar múltiplas cargas de trabalho na mesma máquina física e obter maior utilização de recursos (memória e CPU) - em última análise, reduzindo custos.
# O que está faltando?
No entanto, o que acontece se seu container morrer? Ou pior ainda, o que acontece se a máquina que executa seu container falhar? Os containers não fornecem uma solução para a tolerância a falhas. Ou se você tiver vários containers que precisam da capacidade de se comunicar, como você habilita a rede entre containers? Como isso muda à medida que você cria e derruba containers individuais? A rede de containers pode facilmente se tornar uma bagunça. Por fim, suponha que seu ambiente de produção consista em várias máquinas - como você decide qual máquina usar para executar seu container?
Uma plataforma de orquestração de containers gerencia todo o ciclo de vida de containers individuais, criando e desligando recursos conforme necessário. Se um container for desligado inesperadamente, a plataforma de orquestração reagirá lançando outro container em seu lugar.
Além disso, a plataforma de orquestração fornece um mecanismo para que os aplicativos se comuniquem entre si, mesmo quando containers individuais subjacentes são criados e destruídos.
Por fim, dado (1) um conjunto de cargas de trabalho de container para executar e (2) um conjunto de máquinas em um cluster, o orquestrador de containers examina cada container e determina a máquina ideal para schedular essa carga de trabalho.
Princípios de design
Agora que entendemos a motivação para a orquestração de containers em geral, vamos gastar algum tempo para discutir os princípios motivadores do design por trás de Kubernetes. Ajuda a entender esses princípios para que você possa usar a ferramenta como ela foi destinada a ser usada.
# Declarativo
Talvez o princípio de design mais importante em Kubernetes seja que simplesmente definimos o estado desejado do nosso sistema e deixamos a automação kubernetes trabalhar para garantir que o estado real do sistema reflita esses desejos. Isso tira responsabilidade do operador a responsabilidade de consertar a maioria das coisas quando elas quebram; você simplesmente precisa declarar como seu sistema deve ser em um estado ideal. Kubernetes detectará quando o estado real do sistema não atender a essas expectativas e ele vai intervir em seu nome para corrigir o problema. Isso permite que nossos sistemas sejam auto-recuperáveis e reajam a problemas sem a necessidade de intervenção humana.
O "estado" do seu sistema é definido por uma coleção de objetos. Cada objeto Kubernetes tem (1) uma especificação na qual você fornece o estado desejado e (2) um status que reflete o estado atual do objeto. Kubernetes mantém uma lista de todas as especificações do objeto e constantemente pesquisa cada objeto, a fim de garantir que seu status seja igual à especificação. Se um objeto não responder, o Kubernetes criarár uma nova versão para substituí-lo. Se o status de um objeto se afastar da especificação, Kubernetes emitirá os comandos necessários para direcionar esse objeto de volta ao estado desejado.
# Distribuído
Para uma determinada escala operacional, torna-se necessário arquitetar suas aplicações como um sistema distribuído. Kubernetes foi projetado para fornecer a camada de infraestrutura para tais sistemas distribuídos, produzindo abstrações limpas para construir aplicações em cima de um cluster de máquinas. Mais especificamente, kubernetes fornece uma interface unificada para interagir com este cluster de tal forma que você não precisa se preocupar em se comunicar com cada máquina individualmente.
![image](https://user-images.githubusercontent.com/41973801/215229363-c9e1be6b-01c2-43d3-8ac7-77237665a71d.png)

