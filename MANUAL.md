# illenium-appearance — Manual

Sistema de aparência do personagem: lojas de roupa, barbearia, tatuagem, cirurgia plástica, vestiários de job/gang e outfits salvos.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Permissões (ACE)](#permissões-ace)
4. [Configuração](#configuração)
5. [Lojas e vestiários](#lojas-e-vestiários)
6. [Outfits de job e gang](#outfits-de-job-e-gang)
7. [Códigos de outfit](#códigos-de-outfit)
8. [Blacklist, peds, tatuagens e tema](#blacklist-peds-tatuagens-e-tema)
9. [Comandos](#comandos)
10. [Integrações](#integrações)
11. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
12. [Localização](#localização)
13. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `ox_lib` | Sim | Callbacks, comandos, menus, TextUI, notificações |
| `oxmysql` | Sim | Persistência das skins e dos outfits |
| `qb-core` **ou** `es_extended` | Sim | O framework é detectado em runtime pelo estado do recurso |
| `ox_target` ou `qb-target` | Não | Só com `Config.UseTarget = true`. Substitui o TextUI por interação no ped da loja |
| `qb-radialmenu` ou o radial do `ox_lib` | Não | Só com `Config.UseRadialMenu = true` |
| `qbx_management` | Não | Menu do patrão no QBox, para gerenciar os outfits do job |
| `rcore_tattoos` | Não | Só com `Config.RCoreTattoosCompatibility = true` |

---

## Instalação

1. Copie a pasta `illenium-appearance` para `resources/`.
2. Importe os SQLs de `sql/` no banco de dados:

   | Arquivo | Tabela |
   |---|---|
   | `sql/playerskins.sql` | `playerskins` — aparência ativa de cada personagem |
   | `sql/player_outfits.sql` | `player_outfits` — outfits salvos pelo jogador |
   | `sql/player_outfit_codes.sql` | `player_outfit_codes` — códigos de compartilhamento de outfit |
   | `sql/management_outfits.sql` | `management_outfits` — outfits de job/gang criados pelo patrão |

3. Adicione ao `server.cfg`:
   ```
   ensure illenium-appearance
   ```
4. Defina o idioma da UI (ver [Localização](#localização)):
   ```
   setr illenium-appearance:locale "pt-BR"
   ```
5. **Conflitos** — não rode junto com `qb-clothing`, `fivem-appearance` ou outro recurso de aparência. Para migrar do `qb-clothing`, use o comando `/migrateskins` (ver [Comandos](#comandos)).

---

## Permissões (ACE)

Duas coisas usam ACE.

**Menu de peds** (`/pedmenu`) — restrito ao grupo de `Config.PedMenuGroup` (padrão `group.admin`):

```
add_ace group.admin command.pedmenu allow
```

**Itens restritos por ACE na UI** — com `Config.EnableACEPermissions = true`, entradas de `shared/peds.lua` e `shared/tattoos.lua` podem ser liberadas por ACE, além de por job, gang ou citizenid. Basta adicionar a chave `aces` na entrada:

```lua
{
    name = "TAT_AR_000",
    label = "Turbulence",
    aces = { "illenium.tattoo.vip" },
    -- ...
}
```

```
add_ace group.vip illenium.tattoo.vip allow
```

O recurso descobre as ACEs existentes rodando `list_aces` periodicamente (`Config.ACEListCooldown`, padrão 1 hora), então uma ACE recém-criada pode levar até um ciclo para ser reconhecida.

---

## Configuração

A configuração principal fica em `shared/config.lua`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Config.Debug` | bool | Sim | Ativa logs de diagnóstico |
| `Config.ClothingCost` | number | Sim | Preço de uso da loja de roupas. Padrão: `100` |
| `Config.BarberCost` | number | Sim | Preço da barbearia. Padrão: `100` |
| `Config.TattooCost` | number | Sim | Preço da tatuagem. Padrão: `100` |
| `Config.SurgeonCost` | number | Sim | Preço da cirurgia plástica. Padrão: `100` |
| `Config.ChargePerTattoo` | bool | Sim | Cobra por tatuagem em vez de taxa única. `TattooCost` vira o preço unitário, sobrescrevível com `cost` em `shared/tattoos.lua` |
| `Config.RCoreTattoosCompatibility` | bool | Sim | Só ative se usar o `rcore_tattoos` |
| `Config.AsynchronousLoading` | bool | Sim | `false` carrega os dados da NUI antes de exibir a UI |
| `Config.UseTarget` | bool | Sim | Usa target no ped da loja em vez do TextUI |
| `Config.TextUIOptions` | tabela | Sim | `position` do TextUI. Padrão: `right-center` |
| `Config.NotifyOptions` | tabela | Sim | `position` das notificações. Padrão: `center-right` |
| `Config.OutfitCodeLength` | number | Sim | Tamanho do código de outfit gerado. Padrão: `10` |
| `Config.UseRadialMenu` | bool | Sim | Abre a UI por menu radial |
| `Config.UseOxRadial` | bool | Sim | Usa o radial do `ox_lib`. Exige `UseRadialMenu = true` |
| `Config.EnablePedsForShops` | bool | Sim | Cria os peds nas lojas |
| `Config.EnablePedsForClothingRooms` | bool | Sim | Cria os peds nos vestiários de job/gang |
| `Config.EnablePedsForPlayerOutfitRooms` | bool | Sim | Cria os peds nas salas de outfit do jogador |
| `Config.EnablePedMenu` | bool | Sim | Registra o comando `/pedmenu` |
| `Config.PedMenuGroup` | string | Sim | Grupo com acesso ao `/pedmenu`. Padrão: `group.admin` |
| `Config.EnableJobOutfitsCommand` | bool | Sim | Registra `/joboutfits` e `/gangoutfits` |
| `Config.ShowNearestShopOnly` | bool | Sim | Mostra apenas o blip da loja mais próxima de cada tipo |
| `Config.HideRadar` | bool | Sim | Esconde o minimapa enquanto a UI está aberta |
| `Config.NearestShopBlipUpdateDelay` | number | Sim | Intervalo em ms de atualização do blip mais próximo. Padrão: `10000` |
| `Config.InvincibleDuringCustomization` | bool | Sim | Deixa o jogador invencível durante a customização |
| `Config.PreventTrackerRemoval` | bool | Sim | Desabilita a seção "Scarf and Chains" quando o jogador está de tornozeleira |
| `Config.TrackerClothingOptions` | tabela | Sim | `drawable` e `texture` que representam a tornozeleira |
| `Config.NewCharacterSections` | tabela | Sim | Seções liberadas na criação do personagem: `Ped`, `HeadBlend`, `FaceFeatures`, `HeadOverlays`, `Components`, `Props`, `Tattoos` |
| `Config.GenderBasedOnPed` | bool | Sim | Deriva o gênero do modelo de ped em vez do dado do framework |
| `Config.AlwaysKeepProps` | bool | Sim | Mantém os props ao trocar de roupa |
| `Config.PersistUniforms` | bool | Sim | Mantém o uniforme de job/gang entre reconexões |
| `Config.OnDutyOnlyClothingRooms` | bool | Sim | Restringe os vestiários a quem está em serviço |
| `Config.BossManagedOutfits` | bool | Sim | Permite ao patrão do job/gang gerenciar os outfits da organização |
| `Config.ReloadSkinCooldown` | number | Sim | Cooldown em ms do `/reloadskin`. Padrão: `5000` |
| `Config.AutomaticFade` | bool | Sim | Faz o degradê automático e esconde a seção "Fade" do cabelo |
| `Config.EnableACEPermissions` | bool | Sim | Liga a checagem de ACE nas entradas de ped e tatuagem |
| `Config.ACEResetCooldown` | number | Sim | Cooldown em ms do reset do cache de ACEs. Padrão: `5000` |
| `Config.ACEListCooldown` | number | Sim | Intervalo em ms entre execuções do `list_aces`. Padrão: 1 hora |
| `Config.DisableComponents` | tabela | Sim | Desliga seções de roupa: `Masks`, `UpperBody`, `LowerBody`, `Bags`, `Shoes`, `ScarfAndChains`, `BodyArmor`, `Shirts`, `Decals`, `Jackets` |
| `Config.DisableProps` | tabela | Sim | Desliga seções de acessório: `Hats`, `Glasses`, `Ear`, `Watches`, `Bracelets` |
| `Config.Blips` | tabela | Sim | Blip por tipo de loja (`clothing`, `barber`, `tattoo`, `surgeon`): `Show`, `Sprite`, `Color`, `Scale`, `Name` |
| `Config.TargetConfig` | tabela | Sim | Ped e opção de target por tipo de local: `model`, `scenario`, `icon`, `label`, `distance` |
| `Config.Stores` | tabela | Sim | Lojas do mapa |
| `Config.ClothingRooms` | tabela | Sim | Vestiários de job/gang |
| `Config.PlayerOutfitRooms` | tabela | Sim | Salas de outfit restritas a citizenids |
| `Config.Outfits` | tabela | Sim | Outfits fixos por job/gang |
| `Config.InitialPlayerClothes` | tabela | Sim | Roupa inicial do personagem novo, por gênero: `Model`, `Components`, `Props` e `Hair` |

---

## Lojas e vestiários

### `Config.Stores`

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `type` | string | Sim | `clothing`, `barber`, `tattoo` ou `surgeon` |
| `coords` | `vector4` | Sim | Posição e heading do ped da loja |
| `size` | `vector3` | Sim | Tamanho da zona quando `usePoly = false` |
| `rotation` | number | Sim | Rotação da zona quando `usePoly = false` |
| `usePoly` | bool | Sim | `true` monta a zona a partir de `points`; `false` usa `size` + `rotation` |
| `points` | `vector3[]` | Sim, se `usePoly` | Vértices da zona poligonal |
| `showBlip` | bool | Não | Sobrescreve a visibilidade de blip definida em `Config.Blips` para aquele tipo |
| `targetModel` | string | Não | Sobrescreve o ped de `Config.TargetConfig` |
| `targetScenario` | string | Não | Sobrescreve o scenario de `Config.TargetConfig` |

### `Config.ClothingRooms`

Vestiário de job ou gang. Mesmos campos de zona, mais `job` (ou `gang`) com o nome da organização. Com `Config.OnDutyOnlyClothingRooms = true`, só entra quem estiver em serviço. O config vem com um vestiário da `police`.

### `Config.PlayerOutfitRooms`

Sala de outfits pessoais restrita a uma lista de `citizenIDs`. Mesmos campos de zona. Vem vazia, com um exemplo comentado.

---

## Outfits de job e gang

`Config.Outfits` é indexado pelo nome do job ou da gang e, dentro dele, por `Male` e `Female`. O config já traz `police`, `realestate` e `ambulance`.

```lua
Config.Outfits = {
    ["police"] = {
        ["Male"] = {
            {
                name = "Short Sleeve",
                outfitData = {
                    ["pants"]   = { item = 24, texture = 0 },
                    ["arms"]    = { item = 19, texture = 0 },
                    ["t-shirt"] = { item = 58, texture = 0 },
                    -- vest, torso2, shoes, accessory, bag, hat, glass, mask, decals, ear
                },
                grades = { 0, 1, 2, 3, 4 },
            },
        },
        ["Female"] = { }
    },
}
```

O campo `grades` limita quais níveis do job enxergam o outfit.

Com `Config.BossManagedOutfits = true`, o patrão cria e apaga outfits da organização em jogo. Esses ficam na tabela `management_outfits`, não no `config.lua`. No ESX, o acesso é pelo comando `/bossmanagedoutfits`; no QBox, pelo menu do `qbx_management`.

---

## Códigos de outfit

Um outfit salvo pode gerar um código de `Config.OutfitCodeLength` caracteres, guardado em `player_outfit_codes`. Outro jogador importa esse código pela UI e recebe o mesmo conjunto de roupas.

---

## Blacklist, peds, tatuagens e tema

| Arquivo | Conteúdo |
|---|---|
| `shared/blacklist.lua` | `Config.Blacklist`: drawables e props bloqueados, separados por `male` e `female`, e por `hair`, `components` (masks, upperBody, lowerBody, bags, shoes, scarfAndChains, shirts, bodyArmor, decals, jackets) e `props` (hats, glasses, ear, watches, bracelets) |
| `shared/peds.lua` | `Config.Peds`: modelos de ped disponíveis na UI. Cada grupo aceita `jobs`, `gangs`, `aces` e `citizenids` para restringir quem enxerga |
| `shared/tattoos.lua` | `Config.Tattoos`: catálogo por zona do corpo (`ZONE_TORSO` e afins), com `name`, `label`, `hashMale`, `hashFemale`, `collection` e, opcionalmente, `cost` e as mesmas restrições de acesso |
| `shared/theme.lua` | `Config.Theme`: temas visuais da UI. `currentTheme` escolhe o tema ativo (o repo vem com `murai`) |

---

## Comandos

| Comando | Permissão | Descrição |
|---|---|---|
| `/pedmenu [playerID]` | `Config.PedMenuGroup` (padrão `group.admin`) | Abre a UI completa de aparência. Sem argumento, abre para si mesmo; com um id, abre para o jogador alvo. Só existe se `Config.EnablePedMenu = true` |
| `/joboutfits` | Todos | Abre os outfits do job. Só existe se `Config.EnableJobOutfitsCommand = true` |
| `/gangoutfits` | Todos | Abre os outfits da gang. Mesma condição |
| `/reloadskin` | Todos | Recarrega a aparência salva. Cooldown de `Config.ReloadSkinCooldown` |
| `/bvida` | Todos | Remove props que ficaram presos no personagem |
| `/bossmanagedoutfits` | Patrão do job (ESX) | Gerencia os outfits da organização |
| `/migrateskins <resourceName>` | `group.god` (QB) / `group.admin` (ESX) | Migra as skins de outro recurso de aparência para o `illenium-appearance` |

---

## Integrações

### ox_target / qb-target

Com `Config.UseTarget = true`, o ped de cada local ganha uma opção de target no lugar do TextUI. Modelos, ícones, labels e distâncias vêm de `Config.TargetConfig`. O recurso escolhe automaticamente entre `ox_target` e `qb-target`.

### Menu radial

Com `Config.UseRadialMenu = true`, a UI é aberta pelo menu radial. `Config.UseOxRadial = true` usa o radial do `ox_lib`; caso contrário, o do `qb-radialmenu`.

### qbx_management

No QBox, o gerenciamento de outfits da organização entra no menu do patrão do `qbx_management`, desde que `Config.BossManagedOutfits = true`.

### rcore_tattoos

Com `Config.RCoreTattoosCompatibility = true`, as tatuagens passam a ser responsabilidade do `rcore_tattoos`.

### qb-clothing

Migração suportada: `/migrateskins <resourceName>` converte as skins existentes para o formato do `illenium-appearance`.

---

## Entrypoints para outros recursos

### Exports de cliente

Abrir a customização completa (é o que o multichar usa na criação do personagem):

```lua
exports['illenium-appearance']:startPlayerCustomization(callback, config)
```

Ler a aparência do ped:

```lua
local model      = exports['illenium-appearance']:getPedModel(ped)
local components = exports['illenium-appearance']:getPedComponents(ped)
local props      = exports['illenium-appearance']:getPedProps(ped)
local headBlend  = exports['illenium-appearance']:getPedHeadBlend(ped)
local features   = exports['illenium-appearance']:getPedFaceFeatures(ped)
local overlays   = exports['illenium-appearance']:getPedHeadOverlays(ped)
local hair       = exports['illenium-appearance']:getPedHair(ped)
local appearance = exports['illenium-appearance']:getPedAppearance(ped)
```

Aplicar aparência no ped:

```lua
exports['illenium-appearance']:setPlayerModel(model)
exports['illenium-appearance']:setPlayerAppearance(appearance)
exports['illenium-appearance']:setPedAppearance(ped, appearance)
exports['illenium-appearance']:setPedComponent(ped, component)
exports['illenium-appearance']:setPedComponents(ped, components)
exports['illenium-appearance']:setPedProp(ped, prop)
exports['illenium-appearance']:setPedProps(ped, props)
exports['illenium-appearance']:setPedHeadBlend(ped, headBlend)
exports['illenium-appearance']:setPedFaceFeatures(ped, features)
exports['illenium-appearance']:setPedHeadOverlays(ped, overlays)
exports['illenium-appearance']:setPedHair(ped, hair, headOverlays)
exports['illenium-appearance']:setPedEyeColor(ped, eyeColor)
exports['illenium-appearance']:setPedTattoos(ped, tattoos)
```

### Eventos de cliente

```lua
-- UI completa de aparência
TriggerClientEvent('illenium-appearance:client:openClothingShopMenu', source)

-- Lojas específicas
TriggerClientEvent('illenium-appearance:client:openClothingShop', source)
TriggerClientEvent('illenium-appearance:client:OpenBarberShop', source)
TriggerClientEvent('illenium-appearance:client:OpenTattooShop', source)
TriggerClientEvent('illenium-appearance:client:OpenSurgeonShop', source)

-- Vestiários e outfits
TriggerClientEvent('illenium-appearance:client:OpenClothingRoom', source)
TriggerClientEvent('illenium-appearance:client:OpenPlayerOutfitRoom', source)
TriggerClientEvent('illenium-appearance:client:openOutfitMenu', source)
TriggerClientEvent('illenium-appearance:client:openJobOutfitsMenu', source)
TriggerClientEvent('illenium-appearance:client:loadJobOutfit', source)

-- Recarregar a skin salva e limpar props presos
TriggerClientEvent('illenium-appearance:client:reloadSkin', source)
TriggerClientEvent('illenium-appearance:client:ClearStuckProps', source)
```

### Callbacks de servidor

```lua
-- Aparência salva do jogador
local appearance = lib.callback.await('illenium-appearance:server:getAppearance', false)

-- Outfits do jogador e da organização
local outfits = lib.callback.await('illenium-appearance:server:getOutfits', false)
local mgmt    = lib.callback.await('illenium-appearance:server:getManagementOutfits', false)

-- Uniforme de job/gang ativo
local uniform = lib.callback.await('illenium-appearance:server:getUniform', false)

-- Códigos de outfit
local code = lib.callback.await('illenium-appearance:server:generateOutfitCode', false, outfitId)
lib.callback.await('illenium-appearance:server:importOutfitCode', false, code)
```

### Routing bucket

Durante a customização, o jogador pode ser isolado em um bucket próprio:

```lua
TriggerServerEvent('illenium-appearance:server:ChangeRoutingBucket')  -- move para um bucket privado
TriggerServerEvent('illenium-appearance:server:ResetRoutingBucket')   -- devolve ao bucket 0
```

---

## Localização

Os idiomas são arquivos Lua em `locales/`, carregados pelo `fxmanifest.lua`. O idioma ativo **não** usa `ox:locale` — vem de uma convar própria:

```
setr illenium-appearance:locale "pt-BR"
```

Se o valor não corresponder a um locale carregado, o recurso cai em `en`.

Idiomas carregados: `ar`, `bg`, `cs`, `de`, `en`, `es-ES`, `fr`, `hu`, `id`, `it`, `nl`, `pt-BR`, `ro-RO`.

Os arquivos `locales/zh-CN.lua` e `locales/zh-TW.lua` existem na pasta mas **não estão listados no `fxmanifest.lua`**, então não são carregados. Para usá-los, adicione-os ao bloco `shared_scripts`.

---

## Estrutura de arquivos

```
illenium-appearance/
├── client/
│   ├── client.lua              — ponto de entrada do cliente, abertura das lojas e da UI
│   ├── common.lua              — helpers compartilhados do cliente
│   ├── zones.lua               — zonas das lojas, vestiários e salas de outfit
│   ├── outfits.lua             — outfits salvos, uniformes e códigos de outfit
│   ├── blips.lua               — blips das lojas, incluindo o modo "loja mais próxima"
│   ├── props.lua               — props do personagem e limpeza de props presos
│   ├── stats.lua               — estatísticas do ped
│   ├── defaults.lua            — valores padrão das seções da UI
│   ├── framework/              — camadas de qb-core e es_extended, e a migração do qb-clothing
│   ├── target/                 — camadas de ox_target e qb-target
│   ├── management/             — gerenciamento de outfits pelo patrão (qb, qbx, esx)
│   └── radial/                 — integração com qb-radialmenu e com o radial do ox_lib
├── game/
│   ├── constants.lua           — ids de componente, prop, overlay e zonas de tatuagem
│   ├── util.lua                — leitura e escrita da aparência do ped (exports)
│   ├── customization.lua       — fluxo de customização e o export startPlayerCustomization
│   └── nui.lua                 — ponte com a UI
├── server/
│   ├── server.lua              — eventos, callbacks e comandos
│   ├── permissions.lua         — descoberta e checagem das ACEs
│   ├── util.lua                — helpers do servidor
│   ├── database/               — acesso às tabelas (skins, outfits, códigos, job grades)
│   └── framework/              — camadas de qb-core e es_extended e as migrações
├── shared/
│   ├── config.lua              — opções, lojas, vestiários, outfits de job/gang, roupa inicial
│   ├── blacklist.lua           — drawables e props bloqueados por gênero
│   ├── peds.lua                — modelos de ped disponíveis na UI
│   ├── tattoos.lua             — catálogo de tatuagens por zona do corpo
│   ├── theme.lua               — temas visuais da UI
│   └── framework/              — detecção de framework compartilhada
├── locales/                    — ar, bg, cs, de, en, es-ES, fr, hu, id, it, nl, pt-BR, ro-RO
│                                 (zh-CN e zh-TW existem mas não são carregados)
├── sql/                        — playerskins, player_outfits, player_outfit_codes, management_outfits
├── web/dist/                   — UI compilada (React)
└── fxmanifest.lua
```
