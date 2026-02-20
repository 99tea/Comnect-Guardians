# 🛡️ COMNEcT Guardians 

> **Módulo de gamificação educativa para Cyber Security com elementos de RPG**  
> Desenvolvido como projeto solo e integrado a um sistema interno corporativo (Orion Connect)

---

## 📋 Índice

- [Sobre o Módulo](#sobre-o-módulo)
- [Stack Tecnológica](#stack-tecnológica)
- [Arquitetura](#arquitetura)
- [Minigames e Desafios](#minigames-e-desafios)
- [Sistema de Gamificação](#sistema-de-gamificação)
- [Modo Externo](#modo-externo)
- [Painel Administrativo](#painel-administrativo)
- [Banco de Dados](#banco-de-dados)
- [Patch Notes](#patch-notes)

---

## Sobre o Módulo

O **COMNEcT Guardians** é um sistema de gamificação educativa desenvolvido para engajar colaboradores em treinamentos de segurança da informação de forma progressiva e interativa. Inspirado em jogos de RPG e plataformas de e-learning, transforma conteúdos de cyber security em uma experiência com progressão real, recompensas e competição saudável.

O módulo foi desenvolvido integralmente por mim como uma extensão de um sistema interno corporativo, cobrindo todas as camadas: modelagem do banco, lógica de negócio, templates, sistema de gamificação e deploy.

### O que o sistema oferece

- Cinco tipos de minigames educativos com mecânicas distintas
- Sistema de progressão RPG com especializações, níveis e árvore de habilidades
- Loja de upgrades com sistema Gacha, raridade de itens e controle de expiração
- Conquistas (insígnias) com bônus passivos permanentes e destaque no perfil
- Missões semanais com recompensas e rastreamento de progresso
- Ranking global e perfil público personalizável
- Modo externo para colaboradores de fora do time principal
- Painel administrativo completo com analytics e controle de conteúdo

---

## Stack Tecnológica

```
Backend      → Python 3.13 / Flask (Blueprints)
ORM          → SQLAlchemy + Flask-Migrate (Alembic)
Banco        → MySQL (PyMySQL)
Auth         → Flask-Login + SessionManager customizado
Frontend     → Jinja2 + Bootstrap 5
JavaScript   → Vanilla JS + Chart.js
Deploy       → Windows Server (Waitress WSGI)
Avatares     → DiceBear API (bottts)
```

### Decisões de Design

**Blueprints Flask** — o módulo Guardians é isolado em seu próprio blueprint (`guardians_bp`), com rotas de jogador e rotas admin separadas em arquivos distintos.

**Decorators personalizados** — controle granular de acesso com `login_required`, `guardian_admin_required` e `externo_required`, garantindo que cada rota só seja acessível pelo perfil correto.

**Macros Jinja2** — componentes reutilizáveis para UI complexa (assistente in-game, galeria de conquistas, gestão de externos) evitando duplicação de HTML entre templates.

**SessionManager customizado** — camada de abstração sobre a sessão Flask para centralizar autenticação e leitura de contexto de usuário.

---

## Arquitetura

```
modules/
└── guardians/
    ├── routes.py                   # Rotas de jogadores e externos
    └── admin_refactored_routes.py  # Rotas do painel admin v2
    └── logic.py  # Rotas com lógica de cáculo de pontos
    └── missions_logic.py  # Rotas com lógicas de missões diárias
    └── my_profile.py  # Rotas da página de meu perfil
    └── password_game_rules.py  # Rotas para minigame de senhas
    └── utils_assistant.py  # Rotas do assitente de tutoriais

templates/
└── guardians/
    ├── page_central.html           # Central de desafios (internos)
    ├── page_central_externa.html   # Central de desafios (externos)
    ├── page_meu_perfil.html        # Perfil completo (internos)
    ├── page_perfil_externo.html    # Perfil simplificado (externos)
    ├── page_ranking_externo.html   # Ranking exclusivo externos
    ├── page_resultado_externo.html # Resultado de quiz externo
    ├── play_minigame_quiz.html     # Interface de quiz (compartilhada)
    ├── admin_guardians.html        # Hub admin principal
    ├── admin_analytics_hub.html    # Hub de analytics
    ├── admin_quiz_analysis.html    # Análise individual de quiz
    └── macros/
        ├── macro_externos.html     # Gestão de externos (modal admin)
        ├── assistant_macro.html    # Assistente in-game
        └── render_trophy.html      # Troféu de ranking

application/
└── models.py                       # Todos os modelos SQLAlchemy do módulo
```

---

## Minigames e Desafios

| Tipo | Descrição | Mecânica |
|------|-----------|----------|
| **Quiz** | Questionários de múltipla escolha sobre cyber security com timer | Score base + bônus multicamada, retake com token |
| **Código (Wordle)** | Adivinhe a palavra secreta relacionada a segurança em N tentativas | Feedback por cor de letra (posição e existência) |
| **Decriptar (Anagramas)** | Desembaralhe palavras e termos técnicos para pontuar | Múltiplas palavras por rodada, pontuação por acerto |
| **Segredo (Cofre de Senhas)** | Crie senhas que atendam a requisitos dinâmicos e progressivos | Requisitos gerados progressivamente, valida força e complexidade |
| **Patrulha Diária** | Mini-jogo diário de adivinhar PIN de 4 dígitos (Mastermind) | Feedback por posição e ocorrência, máximo 10 tentativas por dia |
<img width="1196" height="792" alt="image" src="https://github.com/user-attachments/assets/512f69a3-490d-4fba-bc84-43c3bda65ff4" />
<img width="386" height="207" alt="image" src="https://github.com/user-attachments/assets/a668142c-932c-456a-b0f6-2849a509910e" />

---

## Sistema de Gamificação

### Cálculo de Pontuação (Multicamada)

```
Pontuação Final = Score Base
               + Bônus de Especialização  (% baseado no caminho escolhido)
               + Bônus de Loja            (itens passivos ativos no inventário)
               + Bônus de Conquistas      (insígnias equipadas com bônus)
               + Bônus de Velocidade      (conclusão abaixo do tempo médio)
               + Bônus de Perfeição       (100% de acerto)
               + Bônus de Streak          (multiplicador por dias consecutivos)
```

### Especializações (Caminhos)

Três caminhos com árvores de nível exclusivas, bônus passivos únicos e identidade visual própria:

| Caminho | Código | Foco |
|---------|--------|------|
| 🔵 Defensor | `azul` | Defesa, detecção de ameaças e resposta a incidentes |
| 🔴 Hacker | `vermelho` | Ofensiva, pensamento adversarial e pen testing |
| ⚪ Gestor | `cinza` | Governança, gestão de risco e conformidade |

Troca de caminho sujeita a cooldown configurável (dias) e threshold mínimo de XP, ambos ajustáveis pelo admin via `GlobalSettings`.
<img width="1877" height="842" alt="image" src="https://github.com/user-attachments/assets/19f31330-0d03-4d47-8909-efae391ad220" />

### Progressão e Níveis

- Cada especialização tem sua própria árvore de `NivelSeguranca` com threshold de XP, avatar e título exclusivos por nível
- Progresso visual no perfil com barra de XP e indicador do próximo nível
- Ranking global com posição calculada em tempo real

### Conquistas (Insígnias)

- Desbloqueadas por milestones: quizzes perfeitos, compras na loja, streak, patrulhas, etc.
- Cada insígnia pode ter bônus passivo permanente (`bonus_type` + `bonus_value`)
- O guardião pode equipar até N insígnias em destaque no perfil (configurável)
- Agrupadas por categoria com ordem customizável no admin
<img width="780" height="827" alt="image" src="https://github.com/user-attachments/assets/292aad4e-fc30-4d74-87e7-6b6d268e86b7" />

### Streak e Sequência

- Bônus multiplicador progressivo por dias consecutivos de participação
- Tracker visual semanal no perfil (7 indicadores coloridos por status do dia)
- Streak quebrado por inatividade, com estado `lost` rastreado para analytics
<img width="232" height="74" alt="image" src="https://github.com/user-attachments/assets/77c4322b-94f2-49ef-9262-ac668d41dab3" />

### Missões Semanais

- Sets gerados semanalmente com objetivos variados (quizzes, minigames, patrulha)
- Rastreamento de progresso individual por missão (`current_progress` / `target_value`)
- Recompensa em Guardian Coins ao completar todas as missões do set
<img width="780" height="200" alt="image" src="https://github.com/user-attachments/assets/1d3aa6ac-c768-4c4a-91eb-8945bd5529e0" />

### Tokens de Retake

- Moeda especial obtida por desempenho perfeito (100% em quiz ou minigame)
- Configurável: quantos perfeitos são necessários para gerar 1 token
- Permite refazer um desafio já concluído uma vez
<img width="377" height="152" alt="image" src="https://github.com/user-attachments/assets/80dab65b-f644-49a1-bb90-e6d75ca7ca26" />

### Loja de Upgrades (Guardian Coins)

Sistema de **Gacha Diário** com itens renovados periodicamente:

```
Categorias de itens:
├── Módulos Passivos  → bônus percentuais de XP por tipo de atividade (duração definida)
├── Consumíveis       → tokens de retake para segunda chance nos desafios
└── Cosméticos        → itens visuais para personalização do perfil
```

- **Reroll diário**: atualiza os itens disponíveis com custo crescente (1.5× por reroll)
- **Raridades**: Common, Rare, Epic com chances configuráveis pelo admin
- **Inventário com slots**: limite de módulos passivos ativos simultâneos
- **Expiração real**: `expires_at` calculado em compra com base em `duration_days` do item
<img width="1655" height="794" alt="image" src="https://github.com/user-attachments/assets/b1241cdb-d0b2-4f45-905a-76aa83dfe566" />

---

## Modo Externo

Modalidade simplificada desenvolvida para colaboradores fora do time principal, sem duplicação de infraestrutura — utiliza a mesma base de dados com isolamento lógico via flag `is_externo`.

### Implementação Técnica

| Componente | Solução |
|------------|---------|
| Identificação | Coluna `is_externo` na tabela `usuarios` + flag na sessão Flask |
| Autenticação | Mesmo sistema de login com redirecionamento automático pós-login |
| Rastreamento | Campo `external_user_id` em `quiz_attempts` (`guardian_id` tornado nullable) |
| Avatar | Coluna `avatar_seed` persistida em `usuarios` (DiceBear API) |
| Pontuação | Apenas score base — sem bônus, streak ou especialização |
| Isolamento | Ranking e perfil exclusivos, invisíveis para usuários internos |
| Decorator | `@externo_required` bloqueia acesso de internos às rotas externas |

### Fluxo do Usuário Externo

```
Login
  └─▶ Detecção is_externo na sessão
        └─▶ central_externa (listagem de quizzes disponíveis)
              └─▶ start_quiz_externo
                    └─▶ take_quiz_externo (anti-cópia ativo)
                          └─▶ submit_quiz_externo (calcula só score base)
                                └─▶ page_resultado_externo
                                      ├─▶ ranking_externo
                                      └─▶ meu_perfil_externo
```

### Perfil Externo

- Estatísticas: posição no ranking, pontuação total, quizzes feitos, precisão geral
- Barra de precisão animada via CSS transition
- Gráfico de evolução de pontuação por quiz (Chart.js, scroll horizontal automático)
- Histórico de atividades paginado — 10 entradas por vez com botão "ver mais"
- Mensagem motivacional dinâmica baseada na posição no ranking e precisão
- Edição de perfil via modal inline: nome de exibição e avatar

### Proteção Anti-Cópia nos Quizzes

```javascript
document.addEventListener('copy',        e => e.preventDefault());
document.addEventListener('cut',         e => e.preventDefault());
document.addEventListener('contextmenu', e => e.preventDefault());
document.addEventListener('keydown', e => {
    const blocked = (e.ctrlKey && ['c','x','a','u','s'].includes(e.key.toLowerCase()))
                 || e.key === 'F12'
                 || e.key === 'PrintScreen';
    if (blocked) e.preventDefault();
});
```

---

## Painel Administrativo

Controle completo sobre conteúdo, usuários e métricas, acessível apenas a `guardian_admin` ou `portal_admin`.

| Seção | Funcionalidades |
|-------|-----------------|
| **Gerenciar Guardiões** | Listar e editar perfis; promover/rebaixar admin; resetar tokens e saldo |
| **Gerenciar Conteúdo** | CRUD completo de quizzes, questões, termos, anagramas; importação via CSV |
| **Configurações** | Parâmetros globais via `GlobalSettings`: cooldowns, thresholds de XP, limites de slots |
| **Analytics Hub** | Visão geral, análise individual de quiz e relatório detalhado por guardião |
| **Gestão de Externos** | Reset de temporada (todas as tentativas) e reset individual por usuário e quiz |
| **Feedback Hub** | Visualização e gestão de feedbacks enviados in-game |

### Guardian Skill Index (GSI)

Métrica proprietária calculada para cada guardião combinando taxa de acerto, engajamento, streak e desempenho relativo ao grupo. Resultado: índice de **0 a 1000** exibido em gráfico de distribuição e ranking Top 5 no painel.

### Analytics de Quiz

Para cada quiz, o admin visualiza taxa de acerto por questão e por alternativa, distribuição de votos, respostas individuais por guardião, tempo médio de conclusão e dificuldade relativa ao conjunto.

---

## Banco de Dados

### Modelos do Módulo

```
User               → usuarios               (auth, is_externo, avatar_seed)
Guardians          → guardians              (perfil RPG: score, nível, streak, coins, tokens)
Specialization     → specializations        (caminhos: azul, vermelho, cinza)
NivelSeguranca     → niveis_seguranca       (árvore de níveis por especialização)
Quiz               → quizzes                (desafios com período de ativação e expiração)
Question           → questions              (questões com tipo e pontuação)
AnswerOption       → answer_options         (alternativas com flag is_correct)
QuizAttempt        → quiz_attempts          (tentativas internas e externas)
UserAnswer         → user_answers           (resposta de cada questão por tentativa)
ShopItem           → shop_items             (itens com bônus, raridade, duração)
GuardianPurchase   → guardian_purchases     (compras com expires_at calculado)
Insignia           → insignias              (conquistas com bônus passivos)
AchievementCategory→ achievement_categories (categorias de conquistas ordenadas)
GuardianInsignia   → guardian_insignias     (conquistas desbloqueadas por guardião)
GuardianFeatured   → guardian_featured      (insígnias em destaque, slot_index)
HistoricoAcao      → historico_acoes        (log auditável de ações e pontuações)
QuestSet           → quest_sets             (sets de missões semanais)
Mission            → missions               (missões com progresso e is_completed)
GlobalSettings     → global_settings        (parâmetros globais do jogo via chave-valor)
```

## Sobre o Desenvolvimento

Este módulo foi desenvolvido **integralmente por mim como projeto solo**, dentro de um sistema corporativo maior. Minhas responsabilidades incluíram:

- Modelagem do banco de dados e todas as migrations
- Desenvolvimento de todas as rotas e lógica de negócio (backend Flask)
- Criação de todos os templates e componentes de interface (frontend Jinja2 + JS)
- Design completo do sistema de gamificação e suas mecânicas
- Desenvolvimento do painel admin e das ferramentas de analytics
- Configuração e manutenção do ambiente de produção

---

*Guardians Platform — Módulo de Gamificação — v1.2 — Fevereiro 2026*
