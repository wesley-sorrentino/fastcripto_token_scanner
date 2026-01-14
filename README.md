# fastcripto_token_scanner
Plataforma (Next.js + TypeScript) para rastrear tokens que subiram acima 100% na última hora. 
Aplica filtros de FDV / Market Cap / Liquidity, verifica owner renunciado e Makers (Uniswap/Pancake V2 e Uniswap V3).
Inclui painel visual e página de detalhe por token.


# Fast Cripto — Token Scanner

Plataforma para detectar **tokens recém-criados** que subiram **≥ 100% na última hora**, aplicar filtros técnicos e exibir um **painel visual (tema cripto)** com métricas, LP makers e status do contrato.

> ⚠️ **Aviso**: Este projeto é **técnico** e **não** constitui aconselhamento financeiro. Use os dados **apenas** para análise/pesquisa.

---

## 🔎 O que ele faz

- **Gainers (≥ 100%/1h)**: detecta tokens/pares com forte variação na última hora.
- **Filtros automáticos**:
  - **FDV = Market Cap** e **ambos ≥ US$ 100k**
  - **Liquidity ≥ 20% do Market Cap** **e ≥ US$ 10k**
- **LP makers**:
  - **V2** (Uniswap/Pancake): conta endereços únicos que fizeram **add/remove** via eventos `Mint`/`Burn` do **Pair**.
  - **V3** (Uniswap): combina eventos do **Pool** (`Mint`/`Burn`) **+** eventos do **NonfungiblePositionManager** (`IncreaseLiquidity`/`DecreaseLiquidity`), mapeando `tokenId → pool` com a **Factory** (proxy de atores).
- **Owner renunciado (EVM)**: busca ABI (Etherscan/BscScan) e checa se `owner() == 0x000...`.
- **Detalhe do token**: página com **gráfico maior**, métricas, **holders (estimativa por Transfer logs)**, links (Dexscreener) e lista de pares.

---

## 🧱 Stack

- **Next.js 14** (App Router) + **TypeScript**
- **Ethers v6** (RPC EVM / eventos)
- **Dexscreener** (dados de pares) — via endpoints comunitários
- **Supabase** (opcional) para persistência simples
- **CSS** custom (tema cripto, glassmorphism, neon)

---

## 🗺️ Rotas principais (API)

- `GET /api/gainers`  
  Aplica os filtros: **≥ 100%/1h**, **FDV=MC≥100k**, **Liq≥20% do MC e ≥ 10k**.

- `GET /api/lp-makers-v2/[chain]/[pair]`  
  Conta **LP makers V2** (eventos `Mint`/`Burn` do Pair).

- `GET /api/lp-makers-v3/[chain]/[pool]`  
  Conta **LP makers V3** (Pool `Mint`/`Burn` + NFPM `Increase/Decrease` mapeando `tokenId → pool` via `Factory.getPool`).

- `GET /api/owner/[chain]/[address]`  
  Verifica **owner renunciado** (`owner()` == zero address) via `getabi` (Etherscan/BscScan) + chamada RPC.

- `GET /api/token/[chain]/[address]`  
  Retorna pares (Dexscreener), `owner`, **holdersEstimated** (Transfer logs), links.

> **Chains suportadas no MVP**: `ethereum`, `bsc`, `base`.  
> Para outras L2/L1, inclua `Factory`/`NFPM` corretos na `lib/lpMakersV3.ts`.

---

## 🎛️ UI (Frontend)

- `/dashboard`  
  **Cards** com regras/contagens + **tabela** com queries ajustáveis, **toggle V2/V3** para LP makers, badges e **sparklines**.

- `/token/[chain]/[address]`  
  **Detalhe do token**: gráfico maior, FDV, Liquidity, Δ1h, **owner renunciado**, **holders estimados**, links e lista de pares.

Tema visual:
- Dark navy + **neon teal/purple/magenta**
- Gradientes radiais e **glassmorphism**
- Chips por chain e badges de status

---

## ⚙️ Variáveis de ambiente

Crie um arquivo `.env.local` (dev) e configure no **Vercel** (prod):

```env
ETHERSCAN_KEY=********
BSCSCAN_KEY=********
DEXSCREENER_BASE_URL=https://api.dexscreener.com

RPC_URL_ETHEREUM=https://eth-mainnet.g.alchemy.com/v2/********
RPC_URL_BSC=https://bsc-dataseed.binance.org
RPC_URL_BASE=https://mainnet.base.org

# Opcional (se usar Supabase)
SUPABASE_URL=https://********.supabase.co
SUPABASE_ANON_KEY=********
