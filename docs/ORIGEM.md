# Origem do Fides Protocol

**Documento histórico** — Registro da evolução conceitual que levou à criação do protocolo.

---

## A Jornada

### Fase 1: O App de Transparência

A ideia inicial era simples: criar um aplicativo para mostrar quanto cada político gasta. Transparência pura — dados abertos, visualizações, rankings.

**Problema identificado:** Transparência não impede desvio. O dinheiro já saiu. Mostrar depois não resolve.

### Fase 2: A Lei do Rastreamento

Evolução: propor uma lei que obrigue o rastreamento de cada centavo de imposto, desde a arrecadação até o gasto final.

**Problema identificado:** Rastrear ainda é olhar para trás. O desvio acontece, depois você descobre. Complexidade enorme, resultado duvidoso.

### Fase 3: O Momento do Empenho

Insight crucial: o ponto real de poder não é o pagamento, é a **decisão administrativa** que autoriza o gasto (empenho, contrato, aditivo).

Se você controla esse momento, controla tudo que vem depois.

**Pergunta que mudou tudo:** E se o banco simplesmente não pagasse sem um registro válido?

### Fase 4: A Tese + Prova

O projeto deixou de ser "um sistema" e virou algo maior:

1. **Uma tese**: É tecnicamente possível bloquear desvios antes do dinheiro sair
2. **Uma prova**: Implementação de referência que demonstra a tese
3. **Uma pressão**: Se é possível e não querem, por quê?

> "Se eu provo que é possível, a pergunta passa a ser: por que você não quer?"

---

## A Separação

### Dois Projetos Distintos

1. **Fides Protocol** — O núcleo universal
   - Protocolo puro, aplicável a qualquer governo
   - Idioma: inglês
   - Licença: AGPLv3
   - Autoria: diluída, sem pessoa física exposta

2. **Prumo Real** — A implementação brasileira
   - Aplicação do Fides ao contexto brasileiro
   - Documentação em português
   - Estratégia política e jurídica local

### Por Que Separar?

- O protocolo é universal — serve para qualquer país
- A implementação é local — cada país tem suas leis
- Separar protege o núcleo de captura política
- Permite que outros países implementem sem depender do Brasil

---

## O Nome

### Evolução

1. **PIE** (Protocolo Inegociável de Execução) — Nome original em português
2. **Problema:** "PIE" em inglês significa "torta" — não transmite seriedade
3. **Exploração:** Turnstile, GATE, Airlock, Clarion, Veritas, Candor...
4. **Escolha final:** **Fides**

### Por Que Fides?

*Fides* é latim para **confiança, fé, lealdade**.

Na Roma antiga, Fides era a deusa da confiança — a personificação da palavra dada. Contratos eram selados em seu templo.

O protocolo não cria confiança em pessoas. Ele torna a confiança possível ao fazer a integridade ser estrutural.

### Slogan

**"Built for trust."**

Rejeitamos slogans que soavam paternalistas ("When systems are honest, people can be too" — como se pessoas só fossem honestas se controladas).

"Built for trust" é neutro: construído para que confiança seja possível.

---

## Estratégia de Publicação

### Autoria Diluída

O protocolo não tem "dono". Decisões:

- Conta GitHub separada (email anônimo)
- Organização `fides-protocol` como proprietária
- CONTRIBUTORS.md sem nomes reais
- Licença AGPLv3 garante que ninguém pode capturar

### Por Que Anonimato?

Não é medo. É estratégia:

- Ideias sem rosto são mais difíceis de atacar ad hominem
- O foco fica no protocolo, não na pessoa
- Contribuidores podem se juntar sem exposição
- A ideia sobrevive independente do autor original

### Domínio e Hospedagem

- Fase inicial: GitHub Pages (gratuito, sem registro)
- Domínio `fidesprotocol.com`: registrar depois, se necessário
- Prioridade: o protocolo existir e ser verificável

---

## Princípio Central

> **Sem registro válido, não há pagamento.**

Isso não é transparência. Não é auditoria. É uma **trava mecânica**.

O sistema de pagamento simplesmente se recusa a executar sem autorização válida. Não existe override. Não existe exceção operacional. Não existe "liberar dessa vez".

---

## O Que o Fides NÃO É

| O que parece | O que é |
|--------------|---------|
| Sistema de transparência | Trava de execução |
| Ferramenta de auditoria | Pré-requisito de pagamento |
| Dashboard de gastos | Protocolo de autorização |
| Produto comercial | Norma técnica aberta |
| Solução de IA | Verificação binária simples |

---

## Critério de Sucesso

> Um agente mal-intencionado, mesmo com acesso interno, não consegue executar um pagamento irregular sem deixar rastro verificável.

Se isso for verdade, o protocolo funciona.

---

## Relação Entre Repositórios

```
fides-protocol/fides     (protocolo universal, inglês)
        ↓
        implementa
        ↓
usuario/prumoreal        (implementação brasileira, português)
```

O Prumo Real é UMA implementação do Fides. Podem existir outras:
- `fides-mx` (México)
- `fides-ar` (Argentina)
- `fides-eu` (União Europeia)
- etc.

---

## O Que Este Repositório É

**Fides é APENAS o protocolo.**

- Não é sistema
- Não é aplicação
- Não é produto
- Não é implementação

É uma **especificação técnica** — um documento que define regras. Qualquer governo, empresa ou desenvolvedor pode ler e implementar.

O PIE (Protocolo Inegociável de Execução) do Prumo Real **é** o Fides Protocol, apenas traduzido e renomeado para ser universal.

### Entregáveis deste repositório:

1. `spec/FIDES-v0.1.md` — A especificação (tradução do PIE)
2. `reference/*.md` — Documentos de referência técnica
3. `LICENSE` — AGPLv3
4. `README.md` — Explicação pública

**Nenhum código.** Código fica nas implementações (Prumo Real é uma delas).

---

## Próximos Passos

1. **Fides Protocol**
   - [ ] Criar conta GitHub anônima
   - [ ] Criar organização `fides-protocol`
   - [ ] Publicar especificação FIDES-v0.1.md
   - [ ] Adicionar LICENSE (AGPLv3)
   - [ ] Criar documentação de referência

2. **Prumo Real**
   - [ ] Referenciar Fides como base
   - [ ] Manter documentação brasileira
   - [ ] Desenvolver estratégia política local

---

*Este documento registra decisões tomadas durante a concepção do projeto. Serve como referência histórica e não deve ser alterado retroativamente.*
