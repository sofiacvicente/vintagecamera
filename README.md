# Rolo — câmara com filtros (PWA)

O ecrã é só o visor da câmara, já com o filtro aplicado em tempo real. Escolhes **☀ Dia** ou **☾ Noite**, escolhes o filtro e disparas. A foto sai com o grão, a cor, o flash e o carimbo de data laranja.

## Ficheiros

```
index.html              a app inteira (HTML + CSS + JS + shaders WebGL2)
manifest.webmanifest    nome, ícone e modo standalone ("instalar no ecrã principal")
sw.js                   cache offline
icons/                  ícones da app
```

## Pôr no ar

A câmara ao vivo (`getUserMedia`) só funciona em **https** ou em `localhost`.

- **Testar no computador:** `python3 -m http.server` nesta pasta e abre `http://localhost:8000`.
- **Testar no telemóvel:** GitHub Pages (cria um repositório, envia esta pasta e ativa Pages em *Settings → Pages*). Também podes pôr num subdomínio teu, por exemplo `rolo.sofiacvicente.pt`.
- **Instalar:** no iPhone, abre no Safari, *Partilhar → Adicionar ao ecrã principal*. No Android, abre no Chrome e escolhe *Instalar app*.

## Gestos

- Deslizar para os lados no visor muda de filtro.
- Deslizar para cima ou para baixo ajusta a exposição (±2 EV). Toque duplo repõe.
- ⚡ alterna entre automático (só nos filtros com flash), ligado e desligado.
- `DATA` liga e desliga o carimbo de data.
- O ícone de imagem importa uma foto da galeria e aplica-lhe o filtro (botão REVELAR).

## Os filtros

Cada filtro é só um conjunto de números (no topo do `<script>`, em `FILTERS`). O shader faz sempre os mesmos passos, por esta ordem:

1. **Lente:** `soft` mistura com uma versão desfocada (lente barata). `sharpen` faz o contrário, a nitidez exagerada das digicams.
2. **Exposição e balanço de brancos:** `exposure` em stops, `temp` (quente +, frio −) e `tint`.
3. **Flash simulado:** o centro fica mais claro (`flashGain`) e as bordas caem para escuro (`flashDark`). O tamanho do foco é `flashSize`. O efeito adapta-se à luz da cena: num sítio já claro quase não se nota.
4. **Cor do filme:** `mix` é uma matriz 3×3 de mistura de canais (por exemplo, empurra os azuis para ciano no Azul). Depois vem `sat`.
5. **Contraste:** `contrast` linear, `curve` em S, `crush` esmaga os pretos e `shoulder` decide entre ombro suave de filme (1) e corte seco digital (0).
6. **Split toning:** `sh` tinge as sombras e `hi` tinge os realces.
7. **Halation e bloom:** as luzes fortes (acima de `haloThresh`) passam por uma textura pequena, são desfocadas e somadas como halo vermelho (`halation`, como no CineStill 800T) ou como brilho (`bloom`).
8. **Base do filme:** `black` e `white` (RGB). Os pretos nunca são pretos e os brancos nunca são brancos. É isto que dá o ar desbotado.
9. **Vinheta** (`vignette`) e **cantos escuros arredondados** (`corner`, como no filtro Pinhal).
10. **Grão:** `grain` é a quantidade, `grainSize` o tamanho e `chroma` a parte colorida (ruído de cor de CCD). O grão é mais forte nos meios-tons e muda a cada fotograma no visor.

| Filtro | Inspiração | O que o define |
|---|---|---|
| Azul | Fuji Superia 400 a pleno sol | Saturação alta, azuis puxados para ciano, sombras azuladas, grão fino |
| Pinhal | Instant / digital antiga | Quente e desbotado, suave, cantos escuros arredondados |
| Ouro | Kodak Gold 200 | Dourados quentes, realces cremosos |
| Dispo | Descartável com flash | Flash direto, pretos levantados, meios-tons frios, grão forte, vinheta |
| Digicam | Compacta CCD dos anos 2000 | Flash forte com fundo a cair para preto, pele quente, nitidez exagerada, ruído de cor, 4:3 e 2304×1728 (4 MP) |
| Néon | CineStill 800T | Tons frios, halo vermelho à volta das luzes |

Os valores de partida foram tirados das fotos de exemplo: pretos, brancos, cor das sombras e dos realces, saturação, vinheta e grão de cada uma. Ajusta à vontade.

### Acrescentar um filtro

Copia um dos objetos em `FILTERS`, muda `id`, `name`, `mode` (`'dia'` ou `'noite'`), `aspect` (3/2 para 35 mm, 4/3 para digicam) e os números em `p`. Os campos que não puseres ficam com o valor neutro de `BASE`. `flashAuto: true` liga o flash no modo automático. `cap` limita a resolução final.

## Flash

- **Câmara da frente:** o ecrã fica branco no momento da foto (flash de ecrã).
- **Câmara de trás:** a app tenta ligar a lanterna (`torch`). No Chrome para Android funciona. No iPhone depende da versão do iOS. Se não funcionar, fica só o flash simulado do filtro, que já dá o look.

## Onde ficam as fotos

As fotos ficam guardadas no próprio browser (IndexedDB), no "rolo". Para as passares para a galeria, abre a foto e carrega em **Guardar / partilhar**. No iPhone, escolhe *Guardar imagem* na folha de partilha.
