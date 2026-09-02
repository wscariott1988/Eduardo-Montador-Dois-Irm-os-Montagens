# Receita PageSpeed 100/100 — BS Montagem (para replicar)

## O que foi feito (resumo das técnicas)

1. **CSS Crítico inline** — só o CSS da dobra inicial (header, hero, botões, media queries) vai inline no `<head>`. O `style.css` completo carrega assíncrono com `rel="preload" as="style" onload` + fallback `<noscript>`. Zero bloqueio de renderização.
2. **Fontes auto-hospedadas** — Inter e Outfit em WOFF2 variável (48KB e 32KB), sem requisição ao Google Fonts. `font-display:optional` (não causa FOIT nem CLS).
3. **LCP otimizado** — imagem hero em WebP comprimida (12KB desktop / 6.9KB mobile), `fetchpriority="high"` + `<link rel="preload" as="image">` com `srcset` responsivo, e atributos `width`/`height` explícitos (420x525) para zerar CLS.
4. **Google Analytics/Ads atrasado** — `gtag.js` só carrega na 1ª interação do usuário (`click`/`touchstart`). As funções `gtag()` e `gtag_report_conversion()` ficam como stub inline, então conversões funcionam sem custo de performance.
5. **SVG inline em tudo** — ícones (WhatsApp, checks, telefone) e favicon como SVG data-URI. Nenhuma requisição extra, nenhuma fonte de ícone.
6. **Zero dependências externas** — a única requisição externa é o gtag.js, que nem carrega no LCP. O restante (CSS, fontes, imagens) é tudo local.
7. **SEO/Social** — Open Graph + Twitter Card com imagem absoluta.
8. **Clean URLs** — `vercel.json` com rewrites (`/canoas` → `canoas.html`).

---

## PROMPT PARA COLAR NO PRÓXIMO OPENCODE

```
Crie uma landing page estática de alta conversão para [NEGÓCIO] com nota
100/100 no Google PageSpeed (mobile e desktop). Siga EXATAMENTE estas
regras de performance, na ordem de importância:

## ARQUITETURA
- HTML puro + CSS puro, sem frameworks, sem build, sem npm, sem JS externo.
- Um `index.html` + um `style.css` + pasta `Public/` para assets.

## HEAD OBRIGATÓRIO (copie a estrutura abaixo, trocando os dados)
1. Meta charset UTF-8 + viewport.
2. SEO: title, description, keywords.
3. Open Graph completo (og:title, og:description, og:type, og:url, og:image
   com URL absoluta + width/height/type, og:locale, og:site_name).
4. Twitter Card (summary_large_image).
5. Favicon SVG inline como data-URI (nunca arquivo externo).
6. PRELOAD da imagem hero (LCP):
   <link rel="preload" as="image" fetchpriority="high"
         href="Public/Hero.webp"
         imagesrcset="Public/Hero-280.webp 280w, Public/Hero.webp 420w"
         imagesizes="(max-width: 768px) 280px, 420px">
7. Fontes auto-hospedadas: <link rel="preload" href="Public/fonts/Inter.woff2"
   as="font" type="font/woff2" crossorigin> (e Outfit idêntico).
8. CSS CRÍTICO INLINE no <style>: @font-face com font-display:optional
   apontando para os woff2 locais, reset, variáveis CSS, e TODOS os estilos
   da dobra inicial (header fixo, hero, botões CTA, badge, media queries
   de 768px e 480px do hero).
9. CSS completo assíncrono:
   <link rel="preload" href="style.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
   <noscript><link rel="stylesheet" href="style.css"></noscript>

## IMAGENS (regra de ouro)
- Toda imagem em WebP, comprimida ao máximo (hero deve ficar < 15KB).
- SEMPRE gerar 2 versões responsivas: -280.webp (mobile) e .webp (desktop).
- SEMPRE usar <picture> com <source srcset> + <img> com fetchpriority="high"
  na hero, width e height explícitos (proporção 4/5), alt descritivo.
- Nunca imagens JPG/PNG soltas na página.

## ÍCONES E FAVICON
- Todos os ícones como <svg> inline (fill/stroke currentColor).
- Favicon como data:image/svg+xml inline.

## GOOGLE ANALYTICS / ADS (padrão que preserva 100/100)
- No <head>: <link rel="preconnect" href="https://www.googletagmanager.com">
- No <head>: stub do dataLayer + função gtag() + função
  gtag_report_conversion(url) com o ID de conversão do AdWords, sem carregar
  script nenhum.
- No fim do <body>: carregar gtag.js SOMENTE na 1ª interação:
  function loadGTag() { cria <script src="https://www.googletagmanager.com/
  gtag/js?id=AW-XXXXX" async>, onload chama gtag('js',...) e
  gtag('config', 'AW-XXXXX'); }
  document.addEventListener('click', loadGTag, { once: true });
  document.addEventListener('touchstart', loadGTag, { once: true });
- Todos os links de WhatsApp chamam onclick="gtag_report_conversion();" e
  têm target="_blank" rel="noopener".

## SEÇÕES DA PÁGINA (ordem de conversão)
1. Header fixo: logo texto + botão "Orçamento Online" (WhatsApp).
2. Hero: badge de avaliação (★ 5.0 no Google), H1 com cidade/região,
   subtítulo curto, CTA principal WhatsApp (cor chamativa, gradiente),
   texto auxiliar "Resposta rápida no WhatsApp", imagem hero à direita.
3. Diferenciais: 3 cards com ícones SVG (ex.: equipamentos, pontualidade,
   garantia).
4. Serviços: 3 cards com título, descrição e lista de 3 itens com check SVG.
5. Depoimentos: 3 cards com 5 estrelas, texto, avatar com iniciais (div,
   sem imagem), nome e cidade.
6. Footer: marca + descrição, contato (fone/Whats, região, horário),
   link Política de Privacidade.
7. Botão flutuante WhatsApp pulsante fixo no canto.
8. Rodapé com crédito "Desenvolvido por WS Serviços" linkando para
   https://wa.me/5551999402810 com texto pré-preenchido.

## VERIFICAR ANTES DE ENTREGAR
- Nenhum script de analytics carrega no page load (só na interação).
- Nenhuma requisição a Google Fonts ou CDN externo.
- style.css fora do <head> bloqueante (só via preload onload).
- Toda imagem com width/height (zero CLS).
- Testar em https://pagespeed.web.dev — exigir 100/100 mobile e desktop.

## DEPLOY
- Incluir vercel.json com rewrites para URLs limpas:
  { "rewrites": [ { "source": "/NOME", "destination": "/NOME.html" } ] }
- Estrutura final: index.html, style.css, vercel.json, Public/Hero.webp,
  Public/Hero-280.webp, Public/fonts/Inter.woff2, Public/fonts/Outfit.woff2.
```

---

## Assets que você precisa ter (copiar da BS Montagem)

| Asset | Onde conseguir | Peso |
|---|---|---|
| `Public/fonts/Inter.woff2` | Copiar da pasta Public/ deste projeto (varíavel 300-700) | 48KB |
| `Public/fonts/Outfit.woff2` | Copiar da pasta Public/ deste projeto (varíavel 400-800) | 32KB |
| Hero WebP | Converter a foto do cliente em https://squoosh.app (WebP, qualidade ~60, 420x525 e 280x350) | < 15KB |

## Como comprimir a imagem hero (receita exata)

1. Abra https://squoosh.app com a foto.
2. Formato WebP, qualidade 60, resize para **420x525** (desktop).
3. Salve como `Public/Hero.webp`.
4. Repita com resize **280x350** → `Public/Hero-280.webp`.
5. Confirme no Explorer que o desktop ficou < 15KB e o mobile < 8KB.