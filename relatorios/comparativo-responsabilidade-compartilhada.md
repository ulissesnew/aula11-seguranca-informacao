# Comparativo - responsabilidade compartilhada em armazenamento na nuvem

## Escopo

Este relatório considera os dois recursos previstos na atividade: uma **Azure Storage Account** criada no Resource Group `rg-curso-isi` e um **bucket Amazon S3**. O provedor protege a infraestrutura que opera o serviço; o cliente continua responsável por dados, identidades, permissões e pelas configurações que escolhe aplicar.

| Controle | Azure Storage Account - responsabilidade do provedor | Azure Storage Account - responsabilidade do cliente | Bucket Amazon S3 - responsabilidade do provedor | Bucket Amazon S3 - responsabilidade do cliente |
| --- | --- | --- | --- | --- |
| Infraestrutura física e disponibilidade do serviço | Opera datacenters, hardware, rede física, virtualização e o serviço Azure Storage. | Escolher região, redundância e arquitetura adequadas ao requisito de disponibilidade; monitorar o uso do recurso. | Opera datacenters, hardware, rede física, virtualização e o serviço Amazon S3. | Escolher a Região, a estratégia de resiliência e o uso correto do bucket; acompanhar a disponibilidade do aplicativo. |
| Criptografia em repouso | Criptografa os dados automaticamente no serviço com chaves gerenciadas pela Microsoft quando essa é a opção usada. | Classificar os dados; decidir se chaves gerenciadas pelo cliente (CMK) são necessárias; conceder acesso ao Key Vault e rotacionar chaves que administra. | Criptografa automaticamente novos objetos em repouso com SSE-S3 por padrão. | Classificar os dados; escolher e configurar SSE-KMS, DSSE-KMS ou criptografia no cliente quando necessário; administrar permissões e rotação das chaves KMS próprias. |
| Criptografia em trânsito | Fornece endpoints HTTPS/TLS e mecanismos para exigir transferência segura. | Manter `Secure transfer required` habilitado, definir TLS mínimo adequado e garantir que aplicações não usem HTTP. | Fornece endpoints HTTPS/TLS para o serviço. | Exigir HTTPS nas políticas do bucket e assegurar que aplicações e integrações não enviem dados sem TLS. |
| Identidade e controle de acesso | Disponibiliza Microsoft Entra ID, Azure RBAC, SAS e chaves de conta como mecanismos de autorização. | Criar e revisar usuários, grupos, managed identities, RBAC e SAS com menor privilégio; preferir Entra ID e evitar compartilhar chaves de conta. | Disponibiliza IAM, políticas de bucket, access points e mecanismos de autenticação/autorização. | Criar e revisar usuários, roles e políticas IAM/bucket com menor privilégio; evitar credenciais permanentes e não conceder acesso público sem necessidade justificada. |
| Exposição pública e rede | Disponibiliza firewall, regras de rede, Private Link, endpoints privados e controles de acesso público. | Restringir redes/IPs permitidos ou desabilitar acesso público; usar private endpoints quando aplicável. Rede privada não substitui autorização. | Disponibiliza Block Public Access, políticas, VPC endpoints e access points. | Manter Block Public Access habilitado; revisar bucket policies, ACLs e access points; restringir acesso de rede quando o cenário usar endpoints privados. |
| Logs, auditoria e detecção | Disponibiliza logs, métricas e integração com ferramentas de monitoramento do Azure. | Habilitar diagnósticos, reter logs pelo período definido, enviar eventos ao SIEM e investigar alertas. | Disponibiliza CloudTrail, logs e serviços de monitoramento da AWS. | Habilitar e reter CloudTrail e os logs necessários; analisar eventos, configurar alertas e tratar descobertas. |
| Proteção contra exclusão e alteração indevida | Disponibiliza versionamento, soft delete, imutabilidade e mecanismos de bloqueio, conforme o serviço usado. | Definir e testar versionamento, soft delete, retenção, imutabilidade, backups e locks de acordo com RPO/RTO e requisitos legais. | Disponibiliza S3 Versioning, Object Lock e recursos de retenção, conforme configurados. | Habilitar e testar versionamento, retenção, Object Lock quando aplicável e uma política de recuperação para exclusões ou ransomware. |
| Conteúdo, conformidade e resposta a incidentes | Protege a camada do serviço e mantém as certificações e controles aplicáveis à sua infraestrutura. | Responder pela legitimidade do conteúdo, classificação, privacidade, retenção, configuração, uso das credenciais e resposta aos incidentes envolvendo sua conta. | Protege a camada do serviço e mantém controles e certificações aplicáveis à infraestrutura AWS. | Responder pelo conteúdo, classificação, privacidade, retenção, configuração, identidades e resposta aos incidentes da conta AWS. |

## Configuração mínima recomendada para o laboratório

### Azure Storage Account

- Usar Microsoft Entra ID e Azure RBAC com papéis no menor escopo possível.
- Manter transferência segura obrigatória e TLS 1.2 ou superior.
- Restringir o acesso de rede por firewall ou private endpoint quando o cenário permitir.
- Não distribuir chaves da conta; se for inevitável, armazená-las em Azure Key Vault e rotacioná-las.
- Habilitar logs de diagnóstico e uma política de recuperação adequada aos dados armazenados.

### Amazon S3

- Manter **Block Public Access** habilitado, como solicitado no enunciado.
- Manter Object Ownership como `Bucket owner enforced`, salvo uma necessidade comprovada de ACLs.
- Conceder acesso por IAM e bucket policy com menor privilégio e exigir HTTPS.
- Usar SSE-S3 como base; adotar SSE-KMS se houver requisito de controle, auditoria ou segregação de chaves.
- Habilitar versionamento e logs/auditoria compatíveis com a criticidade dos objetos.

## Conclusão

Nos dois serviços, o provedor é responsável por operar e proteger a infraestrutura subjacente e os mecanismos nativos. A proteção efetiva dos dados, porém, depende da configuração do cliente: permissões excessivas, uma política pública, uma chave mal administrada ou logs desabilitados continuam sendo responsabilidade de quem criou e opera a Storage Account ou o bucket.

## Referências

- [Microsoft Learn - Secure an Azure Storage account](https://learn.microsoft.com/en-us/azure/storage/common/secure-storage)
- [Microsoft Learn - Azure Storage encryption for data at rest](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption)
- [Microsoft Learn - Azure Storage network security](https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security-overview)
- [AWS - Protecting data with encryption in Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html)
- [AWS - Granting public access to Amazon S3 data](https://docs.aws.amazon.com/AmazonS3/latest/userguide/granting-public-access.html)
- [AWS - Access control in Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/access-control-overview.html)
