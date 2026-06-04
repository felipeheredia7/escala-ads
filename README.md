# MEF · Extrator de Vídeos da Biblioteca de Anúncios

Ferramenta web que extrai, faz preview, baixa e **transcreve** vídeos da Biblioteca de
Anúncios do Facebook. É um site estático — um único `index.html`, sem servidor e sem
build. Roda inteiro no navegador.

## O que faz

- Cola o link do anúncio, o HTML do `<video>` ou os links `.mp4` diretos → extrai todos os vídeos.
- Preview, **Baixar**, **Abrir**, **Copiar link** e **Baixar todos**.
- **Transcrever**: roda o modelo Whisper localmente no navegador (sem chave de API, sem custo, privado). Saída em português, com copiar texto e baixar `.txt`.
- Bookmarklet "Pegar vídeos MEF" para coletar os links em 1 clique dentro do Facebook.

## Estrutura

```
mef-video-downloader/
├── index.html      # a aplicação inteira (HTML + CSS + JS)
├── vercel.json     # configuração de hospedagem estática
├── .gitignore
└── README.md
```

---

## Como subir no GitHub

Você precisa do Git instalado. Dentro da pasta do projeto, no terminal:

```bash
git init
git add .
git commit -m "MEF: extrator de vídeos da Biblioteca de Anúncios"
git branch -M main
```

Crie um repositório vazio em https://github.com/new (pode ser privado), **sem** README,
e então:

```bash
git remote add origin https://github.com/SEU_USUARIO/mef-video-downloader.git
git push -u origin main
```

> Sem terminal? Dá pra usar o GitHub Desktop ou arrastar os arquivos no botão
> "uploading an existing file" da página do repositório novo.

---

## Como hospedar na Vercel

### Opção A — pelo painel (recomendado, sem terminal)

1. Acesse https://vercel.com e faça login com a conta do GitHub.
2. Clique em **Add New… → Project**.
3. Selecione o repositório `mef-video-downloader` e clique em **Import**.
4. Em **Framework Preset**, deixe **Other**. Não há nada para configurar em Build/Output — é estático.
5. Clique em **Deploy**. Em segundos sai uma URL pública (ex.: `mef-video-downloader.vercel.app`).

A cada `git push` para a branch `main`, a Vercel publica a nova versão automaticamente.

### Opção B — pela Vercel CLI

```bash
npm i -g vercel
vercel        # primeira vez: faz login e cria o projeto
vercel --prod # publica em produção
```

---

## Sobre a transcrição (importante)

O modelo Whisper roda **no navegador do usuário** via `transformers.js` (carregado de CDN,
modelo baixado da Hugging Face na 1ª vez e mantido em cache). Por isso:

- Hospedado na Vercel (HTTPS real), o carregamento do modelo costuma funcionar normalmente.
- É **mais lento** que um serviço de nuvem, pois usa a CPU do usuário.
- Há um trade-off de velocidade no seletor de modelo: Tiny (rápido) / Base / Small (preciso).

### Opcional: deixar a transcrição mais rápida (WASM multi-thread)

Para habilitar threads do WebAssembly, o site precisa ser "cross-origin isolated".
Isso exige dois headers — mas atenção: eles podem quebrar o carregamento de recursos
externos sem CORS (fontes, preview de vídeo). Teste antes de manter. Para ativar,
troque o `vercel.json` por:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "cleanUrls": true,
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "Cross-Origin-Opener-Policy", "value": "same-origin" },
        { "key": "Cross-Origin-Embedder-Policy", "value": "require-corp" }
      ]
    }
  ]
}
```

Se algo parar de carregar depois disso, reverta para o `vercel.json` original.

---

## Aviso de uso

Use apenas para anúncios que você tem direito de baixar. A extração a partir **somente**
do link `/ads/library/?id=...` não é feita pelo site (o Facebook bloqueia leitura externa
da página) — para isso existe o bookmarklet, que roda dentro do próprio Facebook.
