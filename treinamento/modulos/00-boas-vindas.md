# Módulo 0: Boas-vindas

<!-- descricao: Quem é a Be Safe Corp, o que é Active Directory e como é o seu laboratório. -->

Primeiro dia na Be Safe Corp. Você foi contratada ou contratado para a equipe de infraestrutura, e a pergunta que todo mundo faz no início é: *como uma empresa controla quem pode entrar em qual computador, abrir qual pasta e usar qual sistema?*

A resposta, em quase toda empresa que usa Windows, é o **Active Directory**.

## O que é o Active Directory

O Active Directory (AD) é o catálogo central da empresa. Ele guarda:

- **quem existe**: pessoas, computadores, impressoras, contas de serviço;
- **como estão organizados**: setores, grupos, camadas de privilégio;
- **o que cada um pode fazer**: permissões e políticas.

Quando a Ana, do RH, liga o computador e digita a senha, quem confere essa senha não é o computador dela. É o **controlador de domínio**, o servidor que guarda o AD.

## Por que isso importa para segurança

Quem controla o AD controla a empresa inteira. Por isso administrar AD não é só criar usuário: é decidir **quem tem privilégio, por quanto tempo e com que rastro**. Nos próximos módulos você vai ver isso com o nome técnico de IAM (gestão de identidade e acesso) e PAM (gestão de acesso privilegiado).

## Seu laboratório

| Máquina | O que é | Endereço |
|---|---|---|
| `DC01` | o controlador de domínio da Be Safe | 10.10.10.10 |
| `WS01` | o computador de uma pessoa da empresa | 10.10.10.20 |
| `LAB` | este treinamento, o console de correção e as ferramentas de IAM e PAM | 10.10.10.5 |

As três máquinas ficam numa rede isolada: nada do que você fizer aqui sai para a internet ou afeta o seu computador.

!!! tip "Snapshot antes de tudo"
    Antes de começar, tire um **snapshot** das três VMs. Se algo quebrar, você volta ao estado inicial em segundos. Quebrar coisas faz parte do aprendizado.

## Checagem

1. Entre no `DC01` e abra o **Gerenciador do Servidor**. Qual é o nome do domínio que aparece?
2. No `WS01`, abra o PowerShell e rode `whoami`. O que aparece antes da barra invertida, e o que isso significa?

??? note "Respostas"
    1. `besafe.local`.
    2. `BESAFE\`: é o nome curto do domínio. Significa que você entrou com uma conta do AD, e não com uma conta local do computador.
