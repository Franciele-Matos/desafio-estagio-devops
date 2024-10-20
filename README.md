<h1>Infraestrutura como Código na AWS<h1>

> Análise do mainf.tf enviado

###Análise dos Principais Componentes
	**Provider AWS**:
		Define a região em que todos os recursos AWS serão criados, que é us-east-1.
	
	**Varáveis**:
		projeto e candidato:  armazenam os nomes do projeto e do candidato, para personalizar os recursos criados.
		
	**Chave SSH**:
		O código gera uma chave privada RSA com 2048 bits para permitir o acesso à instância EC2 via SSH e associa essa chave a um 'Key Pair' no AWS.
		
	**VPC (Virtual Private Cloud)**:
		Criação de uma VPC com o CIDR 10.0.0.0/16, para definir a rede privada na AWS. 
		Dentro dessa VPC, é criada uma sub-rede com o CIDR 10.0.1.0/24, que será usada para hospedar a instância EC2.
		
	**Internet Gateway**:
		IGW é criado e associado à VPC, permitindo que os recursos dentro da VPC tenham acesso à internet.
		
	**Roteamento**:
		A tabela de rotas é criada para a VPC, liberando que todo o tráfego com destino à internet seja roteado pelo IGW.
		Então, a tabela de rotas é associada à sub-rede, garatindo que também os recurdos da sub-rede tenham conectividade com a internet.
		
	**Security Group**:
		Criado um grupo de segurança que permite acesso SSH de qualquer lugar.
		
	**Imagem AMI debian 12**:
		Seleciona a versão mais recente do Debian 12 para ser usada na criação da instância EC2
		
	**Instância EC2**:
		Cria uma instância EC2 com 20 GB de armazenamento, associada à sub-rede e ao grupo de segurança, 
		e configura um IP público para permitir a comunicação externa. Ao iniciar, a instância atualiza automaticamente o sistema pelo script.
		
	###Observação: por boas práticas, eria interessante criar um novo arquivo chamado `variables.tf` para organizar melhor as variáveis e evitar que fiquem no arquivo `main.tf`.
		
		
> Análise do mainf.tf modificado

	###Security Group:
		**Acesso registrito**: Acesso SSH é limitado a uma rede VPN, reduzindo a superfície de ataque ao não permitir conexões de qualquer lugar.
		**Ingress**: SSH limitado a VPN, em vez de permitir de qualquer lugar.
		**Egress**: Limitação de tráfego de saída para portas 80 (HTTP) e 443 (HTTPS), mantendo um padrão de segurança ao liberar apenas o que é necessário para a aplicação se comunicar com a web.
		
	Instância EC2:
		**Criptografia**: o volume do armazenamento EBS é criptografado.
		**Atualizações de Segurança**: O script user_data foi modificado para garantir que a instância sempre tenha as últimas correções de segurança assim que for criada.
		# Automação da Instalação do NGINX
		Neste projeto, a instalação do NGINX na instância EC2 é realizada automaticamente durante a inicialização da máquina virtual através de um script `user_data`. Este script é executado assim que a instância é criada, garantindo que o NGINX esteja instalado e em funcionamento imediatamente.
		## Script de Instalação

		O seguinte script é utilizado para instalar e configurar o NGINX:
		```bash
             #bin/bash
             apt-get update -y
             apt-get upgrade -y
             apt-get install nginx -y
             systemctl start nginx
             systemctl enable nginx 
             apt-get dist-upgrade -y 
             apt-get autoremove -y
             EOF````

	OBSERVAÇÃO: 
		O IAM oferece um controle mais refinado sobre as permissões. Com o uso de roles do IAM, é possível conceder à instância EC2 acesso a serviços AWS, sem a necessidade de gerenciar chaves de acesso manualmente.
		Desafio Pessoa: Reconheço a importância do IAM no ambiente, mas eu não consegui organizar o código para integrar o IAM de maneira eficiente. O conhecimento sobre como realizar essa implementação de roles e políticas do IAM é algo que pretendo continuar estudando e aplicar em futuros projetos.
		

Intruções de uso
	
	Pré-requisitos:
		**Terraform**: Instale na sua máquina
		**Conta AWS**
		**AWS CLI**
		**GIT**: Instale o Git para clonar o repositório

	Inicializar e Aplicar a Configuração Terraform
		Clone o repositório
			```bash
			git clone <URL_DO_REPOSITORIO>
			cd <NOME_DA_PASTA_DO_PROJETO>````
		Confirme o caminho do diretório onde o arquivo main.tf está armazenado.
		
		Inicialize o Terraform
			terraform init
		
		VIsualize o Plano de Execução
			terraform Plano
			
		Aplique a Configuração
			terraform apply
			
			
	Em seguida, pode realizar a verificação na console AWS se todos os recursos foram criados conforme esperado.
	
	Caso queira, pode destruir os recursos com o comando do terraform:
		terraform destroy