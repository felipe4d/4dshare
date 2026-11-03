# Estrutura documental 4D no SharePoint

Scripts de provisionamento e modelo documental do SharePoint da 4D Soluções,
construídos para o projeto de adequação à ISO/IEC 20000-1:2018.

Este repositório é a **fonte da verdade do modelo documental**. A estrutura no
SharePoint pode ser reconstruída a partir daqui; o contrário não é verdade.

## Pré-requisitos

| Item | Versão / observação |
| --- | --- |
| PowerShell | 7.4.6 ou superior. Não funciona no Windows PowerShell 5.1. |
| PnP.PowerShell | v3 (validado em 3.4.1) |
| App no Entra ID | Registro próprio, obrigatório desde 09/09/2024 |
| Papel | Administrador do SharePoint — a permissão é delegada |

O `ClientId` vem da variável de ambiente `ENTRAID_APP_ID`. **Nenhum script
contém o ClientId, segredo ou certificado** — tudo entra por parâmetro ou
ambiente.

O nome do tenant aparece nos exemplos de uso do cabeçalho de cada script. Não é
segredo (está em toda URL que o colaborador vê), mas se este repositório algum
dia for público, vale trocar por `<tenant>`.

```powershell
Install-Module PnP.PowerShell -Scope CurrentUser
[Environment]::SetEnvironmentVariable('ENTRAID_APP_ID','<client id>','User')
```

## Ordem de execução

Rodar sempre com `-DryRun` primeiro. Todos os scripts são idempotentes:
reexecutar só acrescenta o que falta.

```powershell
# 1. Repositorio ISO, no site de governanca
.\scripts\Provisionar-RepositorioISO.ps1 -Tenant <tenant> -Etapa Colunas
.\scripts\Provisionar-RepositorioISO.ps1 -Tenant <tenant> -Etapa ContentTypes
.\scripts\Provisionar-RepositorioISO.ps1 -Tenant <tenant> -Etapa Bibliotecas

# 2. Conferir lendo de volta
.\scripts\Verificar-RepositorioISO.ps1 -Tenant <tenant>

# 3. Bibliotecas das areas, no site raiz
.\scripts\Provisionar-AreasIntranet.ps1 -Tenant <tenant>
```

## O que cada script faz

| Script | Função | Alcance |
| --- | --- | --- |
| `Provisionar-RepositorioISO.ps1` | Colunas, content type normativo, 4 bibliotecas da ISO, pastas, vistas | Site `governanca` |
| `Verificar-RepositorioISO.ps1` | Somente leitura. Lê o estado real e compara com o esperado | Site `governanca` |
| `Provisionar-AreasIntranet.ps1` | Colunas leves, content type de área, 13 bibliotecas por área | Site **raiz** apenas |
| `Fechar-BibliotecasRestritas.ps1` | Fecha as bibliotecas restritas até a matriz ser aprovada. Só tira acesso | Site **raiz** apenas |

Cada script confere, antes de conectar e antes de qualquer escrita, se todo
cmdlet e parâmetro que usa existe na versão instalada do PnP — e aborta sem
alterar nada se não bater.

## O que este repositório NÃO contém

- **Permissões.** Nenhum script atribui acesso. A matriz depende de aprovação
  formal antes de ser aplicada.
- **Conteúdo.** As bibliotecas nascem vazias. Migração de documento é trabalho
  à parte.
- **A home da intranet.** Página do SharePoint, ainda não provisionada.
- **Grupos do Entra ID.** Dependem de resolver a lista de colaboradores.

## Decisões registradas

Estas estão aqui para não serem reabertas sem contexto.

**Áreas são bibliotecas, não sites.** A fronteira de permissão no SharePoint é a
biblioteca. Site separado só se paga quando há delegação de administração, Teams
próprio, ciclo de vida independente ou segregação contratual — e por isso
`governanca` e `clientes` têm site, e as áreas não.

**Sem term store.** A coluna `Norma aplicavel` é Escolha múltipla, não metadado
gerenciado. Para quatro valores estáticos o term store só trouxe custo: não
existe `Get-PnPTermStore` no PnP 3.4.1, `New-PnPTerm` exige LCID explícito, a
busca por nome não encontra term set existente, e houve escrita reportando
sucesso sem persistir. Migrar de Escolha para Taxonomy depois exige remapear
documento — decidir antes de haver volume.

**Pasta não organiza.** Em `sgi` e `draft` a criação de pasta está desligada.
Quem organiza é o metadado, e as vistas montam a organização. Pasta continua
válida dentro de biblioteca de trabalho, em dois ou três níveis.

**Permissão em biblioteca, nunca em pasta.** Quebrar herança em biblioteca é
administrável e auditável. Em pasta, destrói a auditabilidade.

**Content type de área é separado do normativo.** Forçar `Codigo`,
`Norma aplicavel` e `Clausula ISO` num arquivo de Marketing não faria sentido.

## Limites técnicos que condicionaram o desenho

- Caminho decodificado completo: máximo **400** caracteres (520 no caminho local
  sincronizado); **255** por nome de arquivo ou pasta
- Caracteres proibidos: `" * : < > ? / \ |`, sem espaço no início ou fim
- Montagem automática no desktop: não recomendada acima de **5.000** arquivos
  por biblioteca, nem em mais de **1.000** dispositivos
- Sincronização: no máximo **300.000** itens por pessoa, somando tudo
- Limite de vista: **5.000** itens, salvo coluna de filtro indexada

## Documentação

A arquitetura, a matriz de permissões e o procedimento de operação vivem como
documentos vivos, fora deste repositório. Quando divergirem, o documento é a
referência para intenção e este repositório é a referência para o que está
efetivamente provisionado.
# 4dshare
