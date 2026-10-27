# Arquitetura do Be Safe AD Lab

## Princípios

1. **Ferramentas reais.** AD, GPO, LAPS e PowerShell de verdade. O treinamento explica e confere; não simula.
2. **Nada da Microsoft é redistribuído.** O instalador baixa as ISOs de avaliação oficiais na máquina do aluno.
3. **Rede isolada.** As VMs só conversam entre si (host-only). A internet só é usada durante a montagem.
4. **Correção automática no AD de verdade.** O console consulta o domínio para saber se o exercício foi feito.
5. **Administração e defesa.** O foco é construir e proteger. Ataques aparecem só como motivo para cada controle.

## Topologia

```
        Máquina do aluno (VirtualBox ou VMware)
 ┌──────────────── rede host-only 10.10.10.0/24 ────────────────┐
 │                                                              │
 │  DC01 10.10.10.10        WS01 10.10.10.20     LAB 10.10.10.5 │
 │  Windows Server 2022     Windows 11           Debian         │
 │  AD DS, DNS, GPO         membro do domínio    treinamento    │
 │  Windows LAPS            estação da usuária   console        │
 │                                               Keycloak (IAM) │
 │                                               Guacamole (PAM)│
 └──────────────────────────────────────────────────────────────┘
```

O aluno abre o treinamento no navegador da máquina dele (`http://10.10.10.5`) e trabalha nas VMs Windows.

## Fluxo de montagem

1. **`instalar/`**: script para Windows (PowerShell) e Linux (bash). Confere requisitos, baixa as ISOs de avaliação, confere os hashes e chama o Packer.
2. **`packer/`**: gera as três imagens base (instalação desacompanhada com `autounattend.xml` no Windows e preseed no Debian).
3. **`ansible/`**: configura tudo depois do boot:
   - promove o `DC01` e cria a floresta `besafe.local`;
   - cria a estrutura da empresa (OUs, usuários, grupos) a partir de `group_vars/all.yml`;
   - coloca o `WS01` no domínio;
   - sobe Keycloak, Guacamole, treinamento e console no `LAB`.
4. **Senhas por cópia.** Cada montagem gera senhas novas e mostra ao aluno no final. Nada de senha fixa no repositório.

## IAM

- **Keycloak** com federação LDAP no `DC01`: o AD continua sendo a fonte da verdade das identidades.
- **Portal Be Safe**: app interno simples atrás do Keycloak, para mostrar SSO e MFA (TOTP).
- **Ciclo de vida:** o console simula pedidos do RH (admissão, mudança de setor, desligamento). O aluno executa e o console confere se o acesso ficou certo, inclusive o que deveria ter sido removido.
- **Revisão de acessos:** relatório de quem está em cada grupo sensível, para o aluno encontrar acessos que sobraram.

## PAM

- **Modelo de camadas:** Tier 0 (DC e identidade), Tier 1 (servidores), Tier 2 (estações). OUs e GPOs separadas por camada.
- **Windows LAPS:** senha do administrador local única e rotativa em cada máquina.
- **Protected Users e contas administrativas separadas:** `mariana` para o dia a dia, `adm-mariana` só para administrar.
- **gMSA** para contas de serviço.
- **JIT:** acesso privilegiado temporário por grupo com prazo de validade (recurso PAM do AD, nível funcional 2016).
- **Bastion com Apache Guacamole:** o acesso administrativo ao `DC01` passa pelo bastion e a sessão é gravada.

## Console de correção

Serviço no `LAB` que consulta o AD por LDAP (conta de leitura) e o Keycloak por API. Cada exercício é uma verificação declarativa, por exemplo:

```yaml
id: gpo-senha-financeiro
pergunta: A política de senha está aplicada na OU Financeiro?
verifica:
  tipo: gpo_vinculada
  gpo: "Senha Forte"
  ou: "OU=Financeiro,OU=Be Safe,DC=besafe,DC=local"
```

## Identidade visual

Be Safe Academy: preto e rosa neon, com a pantera ([branding/pantera.webp](../branding/pantera.webp)). Paleta usada no treinamento:

| Nome | Cor |
|---|---|
| preto | `#050407` |
| superfície | `#120a12` |
| rosa | `#fc13b4` |
| rosa claro | `#ff6fd0` |
| texto | `#f5e9f2` |

## Decisões em aberto

- Hipervisor padrão: VirtualBox (gratuito em todos os sistemas) com VMware como alternativa.
- Módulo opcional de ataque e detecção, entre o PAM e a auditoria.
