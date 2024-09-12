<h1>Implantação e Configuração do Kubernetes Dashboard em Ambientes k3s.</h1>

<p>A instalação do Kubernetes Dashboard em um cluster Kubernetes gerenciado pelo k3s é uma maneira eficiente de monitorar e gerenciar os recursos do cluster de forma gráfica. Neste texto, serão abordados os passos para realizar a instalação do Dashboard e sua configuração, permitindo acesso seguro e controle administrativo sobre os componentes do cluster.</p>

<h2>Índice</h2>
<ul>
    <li><a href="#pré-requisitos">Pré-requisitos</a></li>
    <li><a href="#instalação_kubernetes_dashboard"> Instalação do Kubernetes Dashboard</a></li>
    <li><a href="#expondo_via_proxy">Expondo via proxy:</a></li>
    <li><a href="#expondo_via_loadBalancer">Expondo via LoadBalancer:</a></li>

  
    <li><a href="#configuração-do-flannel">Configuração do Flannel</a></li>
    <li><a href="#configuração-do-metallb">Configuração do MetalLB</a></li>
    <li><a href="#deploy-de-aplicações">Deploy de Aplicações</a></li>
    <li><a href="#monitoramento-e-acesso-ao-cluster">Monitoramento e Acesso ao Cluster</a></li>
    <li><a href="#recursos-adicionais">Recursos Adicionais</a></li>
    <li><a href="#contribuições">Contribuições</a></li>
    <li><a href="#licença">Licença</a></li>
</ul>

<h2 id="pré-requisitos">Pré-requisitos</h2>
<ol>
    <p>Para seguir com a instalação, é necessário ter um cluster Kubernetes com k3s já configurado e operando corretamente. No caso deste tutorial, considera-se que o cluster está rodando em um ambiente Raspberry Pi, uma escolha comum para setups      domésticos ou de laboratório. Além disso, deve-se garantir o seguinte:</p>
    <ul>
        <li>k3s instalado e em funcionamento;</li>
        <li>Acesso ao cluster via kubectl configurado;</li>
        <li>Acesso à internet para baixar os manifestos do Kubernetes Dashboard.</li>
    </ul>
</ol>

<h2 id="instalação_kubernetes_dashboard"> Instalação do Kubernetes Dashboard</h2>
<ol>
     <h3>1. Baixando e aplicando o manifesto do Kubernetes Dashboard</h3>
     <p>O Kubernetes Dashboard pode ser instalado através de um manifesto YAML oficial disponibilizado pela comunidade Kubernetes. Para instalá-lo, execute o seguinte comando no seu terminal, que baixará e aplicará o manifesto diretamente no 
         cluster:</p>
     <pre><code>kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml</code></pre>
     <p>Esse comando criará todos os componentes necessários para o Dashboard, como os deployments, serviços e namespaces.</p>    
     <h3>2. Expondo o Kubernetes Dashboard</h3>    
     <p>Por padrão, o Dashboard não está acessível externamente por questões de segurança. Para acessá-lo fora do cluster, podemos expô-lo temporariamente via proxy ou configurar um serviço LoadBalancer.</p>
<!--</ol><-->
    <h3 id="expondo_via_proxy">Expondo via proxy:</h3>
        <ol>
            <p>Uma maneira rápida e fácil de acessar o Dashboard é utilizando o comando proxy do <code>kubectl</code>:</p>
            <pre><code>kubectl proxy</code></pre>
            <p>A partir disso, o Dashboard poderá ser acessado localmente através do URL: <a href="http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/" 
            target="_blank">http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/</a></p> 
        </ol>
    <br>
    <h3 id="expondo_via_loadBalancer">Expondo via LoadBalancer:</h3>
        <ol>
            <p>Caso prefira ter acesso externo ao Dashboard, configure um LoadBalancer. No caso do k3s, o MetalLB pode ser usado para fornecer IPs externos.</p>
            <p>Crie um arquivo <code>service.yaml</code> com o seguinte conteúdo:</p>    
           <pre><code>apiVersion: v1
           kind: Service
           metadata:
             labels:
               k8s-app: kubernetes-dashboard
             name: kubernetes-dashboard-lb
             namespace: kubernetes-dashboard
           spec:
             ports:
               - port: 443
                 targetPort: 8443
             selector:
               k8s-app: kubernetes-dashboard
             type: LoadBalancer
           </code></pre>    
        <p>Aplique o serviço:</p>
        <pre><code>kubectl apply -f service.yaml</code></pre>    
        <p>Esse serviço agora estará acessível externamente no IP atribuído pelo LoadBalancer.</p>
       </ol>
</ol>

<h2>4. Autenticação no Kubernetes Dashboard</h2>

<p>Após configurar o acesso ao Dashboard, será necessário autenticar-se para utilizá-lo. No Kubernetes, a autenticação é feita utilizando tokens de acesso. Para gerar um token de administrador, execute os seguintes comandos:</p>
</ol>





<pre><code>curl -sfL https://get.k3s.io | sh -s - --flannel-iface=eth0
</code></pre>
<ol start="2">
    <li>Obtenha o token do master para conectar os nós workers:</li>
</ol>
<pre><code>cat /var/lib/rancher/k3s/server/node-token
</code></pre>
<ol start="3">
    <li>Conecte os workers ao cluster:</li>
</ol>
<pre><code>curl -sfL https://get.k3s.io | K3S_URL=https://&lt;master-node-ip&gt;:6443 K3S_TOKEN=&lt;token&gt; sh -
</code></pre>

<h2 id="configuração-do-flannel">Configuração do Flannel</h2>
<p>O Flannel é configurado automaticamente pelo k3s. Verifique se a rede está funcional com:</p>
<pre><code>kubectl get pods -A
</code></pre>

<h2 id="configuração-do-metallb">Configuração do MetalLB</h2>
<ol>
    <li>Instale o MetalLB:</li>
</ol>
<pre><code>sudo kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
</code></pre>
<ol start="2">
    <li>Configure o MetalLB com um intervalo de IPs da sua rede local:</li>
</ol>
<pre><code>apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: ip-pool
spc:
  address:
    - 192.168.1.100-192.168.1.110
</code></pre>
<ol start="3">
    <li>Aplique a configuração:</li>
</ol>
<pre><code>kubectl apply -f poll-metallb.yaml
</code></pre>
<ol start="4">
    <li>Configurando Advanced L2 Configuration:</li>
</ol>
<pre><code>apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: homelab-l2
  namespace: metallb-system
spec:
  ipAddressPools:
    - ip-pool
</code></pre>
<ol start="5">
    <li>Aplique a configuração Advanced L2 Configuration:</li>
</ol>
<pre><code>sudo kubectl apply -f L2Advertisement.yaml
</code></pre>

<h2 id="deploy-de-aplicações">Configurando Nginx para teste</h2>
<p>Exemplo de deploy de uma aplicação simples:</p>
<pre><code>sudo kubectl create deploy nginx –image nginx:latest</code></pre>
<p>Expondo o Nginx para acessar pelo browser:</p>
<pre><code>sudo kubectl expose deploy nginx –port 80 --type LoadBalancer</code></pre>

<p>Agora podemos acessar pelo endereço http://192.168.0.101:80 no seu browser</p>



<h2 id="monitoramento-e-acesso-ao-cluster">Monitoramento e Acesso ao Cluster</h2>
<ul>
    <li>Acesse suas aplicações pelo IP atribuído pelo MetalLB.</li>
    <li>Use <code>kubectl</code> para monitorar o estado do cluster e dos pods.</li>
</ul>

<h2 id="recursos-adicionais">Recursos Adicionais</h2>
<ul>
    <li><a href="https://k3s.io/">Documentação oficial do k3s</a></li>
    <li><a href="https://github.com/flannel-io/flannel">Flannel</a></li>
    <li><a href="https://metallb.universe.tf/">MetalLB</a></li>
</ul>

<h2 id="contribuições">Contribuições</h2>
<p>Contribuições são bem-vindas! Sinta-se à vontade para abrir issues e pull requests.</p>

<h2 id="licença">Licença</h2>
<p>Este projeto está licenciado sob a licença MIT. Veja o arquivo <a href="LICENSE">LICENSE</a> para mais detalhes.</p>

