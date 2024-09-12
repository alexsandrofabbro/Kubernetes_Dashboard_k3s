<h1>Implantação e Configuração do Kubernetes Dashboard em Ambientes k3s.</h1>

<p>A instalação do Kubernetes Dashboard em um cluster Kubernetes gerenciado pelo k3s é uma maneira eficiente de monitorar e gerenciar os recursos do cluster de forma gráfica. Neste texto, serão abordados os passos para realizar a instalação do Dashboard e sua configuração, permitindo acesso seguro e controle administrativo sobre os componentes do cluster.</p>

<h2>Índice</h2>
<ul>
    <li><a href="#pré-requisitos">Pré-requisitos</a></li>
    <li><a href="#instalação_kubernetes_dashboard"> Instalação do Kubernetes Dashboard</a></li>
    <li><a href="#autenticacao_kubernetes_dashboard">Autenticação no Kubernetes Dashboard</a></li>
    <li><a href="#conclusao">Conclusão</a></li>   
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
  <ol>
        <h3>2.1Expondo via proxy:</h3>
            <ol>
                <p>Uma maneira rápida e fácil de acessar o Dashboard é utilizando o comando proxy do <code>kubectl</code>:</p>
                <pre><code>kubectl proxy</code></pre>
                <p>A partir disso, o Dashboard poderá ser acessado localmente através do URL: <a href="http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/" 
                target="_blank">http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/</a></p> 
            </ol>
        <br>
        <h3>2.2 Expondo via LoadBalancer:</h3>
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
</ol>

<h2 id="autenticacao_kubernetes_dashboard">Autenticação no Kubernetes Dashboard</h2>

<p>Após configurar o acesso ao Dashboard, será necessário autenticar-se para utilizá-lo. No Kubernetes, a autenticação é feita utilizando tokens de acesso. Para gerar um token de administrador, execute os seguintes comandos:</p>
<ol>
    <h3>1. Crie um usuário admin:</h3>
    <ol>
        <pre><code>apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: admin-user
          namespace: kubernetes-dashboard
        </code></pre>    
       <p>Em seguida, aplique a seguinte configuração de <code>ClusterRoleBinding</code>:</p>
    </ol>
    <h3>2. Gere o token de acesso:</h3>
    <ol>
        <pre><code>kubectl -n kubernetes-dashboard create token admin-user</code></pre>
        <p>Esse comando gerará um token que deverá ser utilizado para se autenticar na interface do Dashboard.</p>
    </ol>
</ol>

<h2 id="conclusao">5. Conclusão</h2>
<ol>
    <p>A instalação do Kubernetes Dashboard em um cluster gerenciado pelo k3s é um processo direto, mas que requer atenção a detalhes relacionados à segurança e acessibilidade. O Dashboard oferece uma           forma eficiente de gerenciar o cluster de maneira visual, sendo uma ferramenta essencial para monitoramento e administração em pequenos e médios ambientes de Kubernetes.</p>
</ol>
