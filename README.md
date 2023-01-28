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

# Desacoplado
É recomendável que os containers sejam desenvolvidos com uma única preocupação em mente (SRP do SOLID). Como resultado, o desenvolvimento de aplicativos conteinerizados se presta muito bem ao padrão de design de arquitetura de microsserviço, que recomenda "projetar aplicativos de software como suítes de serviços independentemente implantáveis".
As abstrações fornecidas em Kubernetes naturalmente apoiam a ideia de serviços desacoplados que podem ser dimensionados e atualizados de forma independente. Esses serviços são logicamente separados e se comunicam através de APIs bem definidas. Essa separação lógica permite que as equipes implantem mudanças na produção em uma velocidade maior, uma vez que cada serviço pode operar em ciclos de releases independentes (desde que respeitem os contratos de API existentes).

# Infraestrutura imutável
Para obter o maior benefício de containers e orquestração de containers, você deve estar implantando infraestrutura imutável. Isto é, em vez de fazer login em um container em uma máquina para fazer alterações (por exemplo, atualizar uma biblioteca), você deve construir uma nova imagem de container, implantar a nova versão e matar a versão mais antiga. À medida que você transita por ambientes durante o ciclo de vida de um projeto (desenvolvimento -> teste -> produção) você deve usar a mesma imagem de container e apenas modificar configurações externas à imagem (por exemplo, montando um arquivo de config).
Isso se torna muito importante, uma vez que os containers são projetados para serem efêmeros, prontos para serem substituídos por outra instância de container a qualquer momento. Se o seu container original tivesse estado mutado (por exemplo, configuração manual), mas fosse desligado devido a uma verificação de saúde falhada, o novo container criado em seu lugar não refletiria essas alterações manuais e poderia potencialmente quebrar sua aplicação.
Quando você mantém uma infraestrutura imutável, também se torna muito mais fácil reverter seus aplicativos para um estado anterior (por exemplo, se ocorrer um erro) - você pode simplesmente atualizar sua configuração para usar uma imagem de container mais antiga.
# Objetos básicos em Kubernetes
Anteriormente, mencionei que descrevemos nosso estado desejado do sistema através de uma coleção de objetos Kubernetes. Até agora, nossa discussão sobre Kubernetes tem sido relativamente abstrata e de alto nível. Nesta seção, vamos investigar mais detalhes sobre como você pode implantar aplicativos em Kubernetes, cobrindo os objetos básicos disponíveis em Kubernetes.
Os objetos Kubernetes podem ser definidos usando arquivos YAML ou JSON; esses arquivos que definem objetos são comumente referidos como manifestos. É uma boa prática manter esses manifestos em um repositório controlado por versão que pode agir como a única fonte de verdade sobre quais objetos estão rodando em seu cluster.
# Pod
O objeto Pod é o bloco de construção fundamental em Kubernetes, composto por um ou mais containers (fortemente relacionados), uma camada de rede compartilhada e volumes de sistema de arquivos compartilhados. Semelhante aos containers, os pods são projetadas para serem efêmeros - não há expectativa de que um pod específico e individual persista por uma longa vida.

![image](https://user-images.githubusercontent.com/41973801/215229458-694b8d6b-9335-4e07-9dfa-c70a37600db1.png)

Você normalmente não criará objetos Pod explicitamente em seus manifestos, pois muitas vezes é mais simples usar componentes de nível mais alto que gerenciam objetos Pod para você.
# Deployment / Deploy
Um objeto de deploy abrange uma coleção de pods definidos por um modelo e uma contagem de réplicas (quantas cópias do modelo que queremos executar). Você pode definir um valor específico para a contagem de réplicas ou usar um recurso Kubernetes separado (por exemplo, um autoescalador de pod horizontal) para controlar a contagem de réplicas com base em métricas do sistema, como a utilização da CPU.
Nota: O deployments na verdade cria outro objeto, um ReplicaSet por baixo dos panos. No entanto, isso é abstraído do usuário.

![image](https://user-images.githubusercontent.com/41973801/215229499-08a681a5-17f7-4245-b974-1b6fec57ba23.png)

Embora você não deva confiar em qualquer pod único para permanecer funcionando infinitamente, você pode confiar no fato de que o cluster sempre tentará ter n pods disponíveis (onde n é definido pela contagem de réplicas especificada). Se tivermos uma deployment com uma contagem de réplicas de 10 e 3 desses pods falhar devido a uma falha na máquina, mais 3 pods serão programadas para funcionar em uma máquina diferente no cluster. Por essa razão, os deploys são mais adequados para aplicações stateless, onde os Pods podem ser substituídos a qualquer momento sem quebrar as coisas.
O seguinte arquivo YAML fornece um exemplo anotado de como você pode definir um objeto de deploy. Neste exemplo, queremos executar 10 instâncias de um container que serve um modelo ML sobre uma interface REST.

![image](https://user-images.githubusercontent.com/41973801/215229537-4d5e6daa-dcf1-4258-bbe4-594754adef57.png)

Nota: Para que kubernetes saiba o quão intensiva em computação essa carga de trabalho pode ser, também devemos fornecer limites de recursos na especificação do modelo de pod.
Os deployments também nos permitem especificar como gostaríamos de implementar atualizações quando tivermos novas versões de nossa imagem de container. Se quiséssemos anular os padrões, incluiríamos um campo adicional sob o objeto. Kubernetes garantirá graciosamente desligar pods executando a imagem antiga do container e girar novos Pods executando a nova imagem do container. strategy spec

![image](https://user-images.githubusercontent.com/41973801/215229554-98084ac4-8f58-4115-88e2-2d3096f3f694.png)

# Service
Cada Pod em Kubernetes recebe um endereço IP exclusivo que podemos usar para se comunicar com ele. No entanto, como os Pods são efêmeros, pode ser bastante difícil enviar tráfego para o container desejado. Por exemplo, vamos considerar o deploy de cima, onde temos 10 Pods executando um container servindo um modelo de machine learning sobre REST. Como nos comunicamos de forma confiável com um servidor se o conjunto de Pods em execução como parte do deployment pode mudar a qualquer momento? É aqui que o objeto service entra em ação. Um Service Kubernetes fornece um endpoint estável que pode ser usado para direcionar o tráfego para os Pods desejados, mesmo que os Pods subjacentes mudem devido a atualizações, escalas e falhas. Os services sabem para quais pods devem enviar tráfego com base em tags (pares de valor de chave) que definimos nos metadados do Pod.

![image](https://user-images.githubusercontent.com/41973801/215229609-d79cba1a-fbc7-4dbc-a998-fa577c68c093.png)

Neste exemplo, nosso Serviço envia tráfego para todos os Pods saudáveis com o rótulo .app="ml-model"
O seguinte arquivo YAML fornece um exemplo de como podemos envolver um Serviço em torno do exemplo de deployment anterior.

![image](https://user-images.githubusercontent.com/41973801/215229637-6f3194d7-3d6f-48ec-8eb2-40e1013290a3.png)
# Ingress
Enquanto um Serviço nos permite expor aplicativos atrás de um endpoint estável, oendpoint só está disponível para o tráfego interno de clusters. Se queríamos expor nossa aplicação ao tráfego externo ao nosso cluster, precisamos definir um objeto Ingress.

![image](https://user-images.githubusercontent.com/41973801/215229671-faac8ad0-7b75-485c-831c-5fbcf67f5090.png)

A vantagem dessa abordagem é que você pode selecionar quais serviços disponibilizar publicamente. Por exemplo, suponha que, além do nosso Serviço para um modelo de machine learning, tínhamos uma interface do usuário que alavancava as previsões do modelo como parte de uma aplicação maior. Podemos optar por disponibilizar apenas a Interface do Usuário para o tráfego público, impedindo que os usuários possam consultar o modelo que serve o Serviço diretamente.

![image](https://user-images.githubusercontent.com/41973801/215229684-423e9947-c15c-4d8f-8722-03f3d528056a.png)

O seguinte arquivo YAML define um objeto Ingress para o exemplo acima, tornando a interface do usuário acessível publicamente.

![image](https://user-images.githubusercontent.com/41973801/215229697-557ee22b-a173-480f-8e62-ccde7884442f.png)
# Job
Os objetos Kubernetes descritos até este ponto podem ser compostos para criar serviços confiáveis e de longa duração. Em contraste, o objeto Job é útil quando você deseja executar uma tarefa discreta. Por exemplo, suponha que queremos re-treinar nosso modelo diariamente com base nas informações coletadas do dia anterior. Todos os dias, queremos subir um container para executar uma carga de trabalho predefinida (por exemplo, um script) e, em seguida, desligar quando o treinamento terminar. Jobs nos fornecem a capacidade de fazer exatamente isso! Se por algum motivo nosso container falhar antes de terminar o script, Kubernetes reagirá criando um novo Pod em seu lugar para terminar o Job. Para objetos de Job, o "estado desejado" do objeto é a conclusão do job. train.py
O YAML a seguir define um exemplo de Job para treinamento de um modelo de machine learning (assumindo que o código de treinamento seja definido em ). train.py

![image](https://user-images.githubusercontent.com/41973801/215229769-5bc889bc-0486-4a6f-9832-8379cf2393c2.png)

Nota: Esta especificação do Job executará apenas uma única execução de treinamento. Se quiséssemos executar este job diariamente, poderíamos definir um objeto CronJob em vez disso.
... e muitos mais.

















