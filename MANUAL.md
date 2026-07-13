# 🎨 illenium-appearance - Manual de Funcionalidades

Sistema flexível e rico em funcionalidades de personalização de jogador para FiveM com suporte para QBCore, ESX e ox_core.

**Versão:** v5.6.1 | **Framework:** QBCore, ESX, ox_core | **Licença:** GPL-3.0

---

## 🎯 O que o illenium-appearance faz

O illenium-appearance é um sistema completo de personalização de jogador (skin/aparência) para FiveM. Ele permite personalização detalhada de aparência (rosto, cabelo, etc.), roupas, tatuagens, modificação de ped (modelo), e gerenciamento de outfits. As skins são salvas persistentemente no banco de dados.

---

## ⚙️ Como funciona

O recurso gerencia a aparência do jogador através de:
- **Cliente:** Interface NUI para personalização visual, aplicação de skins nos peds
- **Servidor:** Persistência de dados no banco de dados, gerenciamento de outfits
- **Configuração:** Definições em `shared/config.lua` e suporte a múltiplos frameworks

A interface web permite personalização em tempo real com preview das mudanças antes de salvar.

---

## 🔧 Configuração

Edite `shared/config.lua`:

```lua
Config = {
    Framework = 'qb-core',        -- 'qb-core', 'esx', 'ox_core'
    EnableTattoos = true,          -- Habilitar sistema de tatuagens
    EnableJobClothing = true,      -- Roupas específicas de emprego
    EnablePedMenu = true,          -- Menu de mudança de ped
    
    -- Blacklist de componentes por job/gang/ACE
    BlacklistedComponents = {
        ['police'] = {
            components = {1, 2, 3},    -- IDs de componentes
            props = {0, 1}            -- IDs de props
        }
    },
    
    -- Temas disponíveis
    Themes = {
        Default = true,
        QBCore = false
    },
    
    -- Salas de roupas restritas
    ClothingRooms = {
        ['police'] = {
            coords = vec3(25.0, -1345.0, 29.5),
            jobs = {'police'}
        }
    }
}
```

### Configuração de Tatuagens
```lua
Config.EnableTattoos = true
Config.TattooPrice = 500            -- Preço por tatuagem
```

### Configuração de Roupas de Emprego
```lua
Config.EnableJobClothing = true
Config.JobOutfits = {
    ['police'] = {
        male = {
            {label = 'Uniforme Padrão', components = {...}, props = {...}}
        },
        female = {
            {label = 'Uniforme Padrão', components = {...}, props = {...}}
        }
    }
}
```

---

## 📤 Exports

### Exports do Cliente
| Export | Descrição | Parâmetros |
|--------|-----------|-------------|
| `setPlayerAppearance` | Define aparência do jogador | `ped` (int), `appearance` (table) |
| `getPlayerAppearance` | Obtém aparência atual | `ped` (int) |
| `openAppearanceMenu` | Abre menu de personalização | `ped` (int) |
| `saveOutfit` | Salva outfit atual | `outfitName` (string), `isJobOutfit` (bool) |
| `loadOutfit` | Carrega outfit salvo | `outfitId` (int) |
| `toggleMenu` | Alterna menu principal | None |

### Exports do Servidor
| Export | Descrição | Parâmetros |
|--------|-----------|-------------|
| `savePlayerSkin` | Salva skin no banco | `src` (int), `skin` (table) |
| `getPlayerSkins` | Recupera skins do jogador | `citizenid` (string) |
| `saveOutfit` | Salva outfit | `src` (int), `outfit` (table) |
| `getOutfits` | Lista outfits do jogador | `citizenid` (string) |

---

## 📡 Eventos

### Eventos do Cliente
| Evento | Descrição | Parâmetros |
|--------|-----------|-------------|
| `illenium-appearance:open` | Abre menu de aparência | `ped` (int) |
| `illenium-appearance:saveSkin` | Salva skin atual | None |
| `illenium-appearance:loadSkin` | Carrega skin salva | `skin` (table) |

### Eventos do Servidor
| Evento | Descrição | Parâmetros |
|--------|-----------|-------------|
| `illenium-appearance:saveOutfit` | Salva outfit | `outfit` (table), `cb` (callback) |
| `illenium-appearance:getOutfits` | Recupera outfits | `src` (int), `cb` (callback) |

---

## 🎮 Comandos

| Comando | Descrição | Permissão |
|---------|-----------|------------|
| `/pedmenu` | Abre menu de personalização de ped | Admin/Configurado |
| `/reloadskin` | Recarrega skin salva do jogador | Todos os Jogadores |

---

## 🔗 Integrações

### Frameworks Suportados
- **QBCore** - Suporte completo (padrão)
- **ESX** - Suporte completo
- **ox_core** - Suporte experimental

### Dependências
- **ox_lib** (obrigatório) - UI e utilitários
- **qb-core** (opcional) - Apenas para QBCore
- **es_extended** (opcional) - Apenas para ESX
- **ox_core** (opcional) - Suporte experimental
- **qb-target** (opcional) - Integração de target

### Integração com qb-target
```lua
-- Criar NPC de cirurgião plástico
exports['qb-target']:AddTargetModel('a_m_m_soucent_03', {
    options = {
        {
            label = 'Alterar Aparência',
            icon = 'fas fa-user-edit',
            action = function()
                TriggerEvent('illenium-appearance:client:openClothingShop')
            end
        }
    }
})
```

### Integração com Polyzone
```lua
-- Zona de sala de roupas
exports['PolyZone']:CreateCircle(vec2(25.0, -1345.0), 2.0, {
    name = 'clothing_store',
    debugPoly = false
})
```

---

## 💡 Casos de Uso

### Definir Aparência do Jogador (Cliente)
```lua
local appearance = {
    model = 'mp_m_freemode_01',
    components = {
        {component_id = 1, drawable = 0, texture = 0},
        {component_id = 2, drawable = 5, texture = 1}
    },
    props = {
        {prop_id = 0, drawable = 2, texture = 0}
    },
    headBlend = {
        shapeFirst = 0, shapeSecond = 0, shapeMix = 0.5,
        skinFirst = 0, skinSecond = 0, skinMix = 0.5
    },
    hair = {
        hairColor = 1, hairHighlight = 0
    }
}

exports['illenium-appearance']:setPlayerAppearance(PlayerPedId(), appearance)
```

### Salvar Outfit do Jogador (Servidor)
```lua
exports['illenium-appearance']:saveOutfit(source, {
    name = 'Uniforme Policial',
    isJobOutfit = true,
    job = 'police',
    components = {...},
    props = {...}
})
```

### Verificar e Aplicar Skin no Spawn (Servidor)
```lua
RegisterServerEvent('playerSpawned', function()
    local src = source
    local citizenid = QBCore.Functions.GetPlayer(src).PlayerData.citizenid
    local skins = exports['illenium-appearance']:getPlayerSkins(citizenid)
    
    if skins and skins[1] then
        TriggerClientEvent('illenium-appearance:loadSkin', src, skins[1])
    else
        TriggerClientEvent('illenium-appearance:open', src, PlayerPedId())
    end
end)
```

### Restringir Roupas por Job
```lua
-- Em config.lua
Config.JobClothing = {
    ['police'] = {
        restrictToJob = true,
        outfits = {...}
    }
}
```

---

## 🎨 Personalização

### Temas Disponíveis
- **Default** - Tema padrão
- **QBCore** - Tema inspirado no QBCore

### Tatuagens
O sistema suporta tatuagens para todos os peds:
- Homens (`mp_m_freemode_01`)
- Mulheres (`mp_f_freemode_01`)
- Peds personalizados

### Migração de Skins
O illenium-appearance suporta migração de:
- qb-clothing
- esx_skin
- Versões anteriores do fivem-appearance

---

## ⚠️ Solução de Problemas

### Skin não carrega no spawn
- Verifique se o banco de dados tem a tabela correta
- Confirme que o citizenid está correto
- Verifique se o recurso tem permissão para ler o banco

### Tatuagens não aparecem
- Verifique se `Config.EnableTattoos = true`
- Confirme que os overlays de tatuagem estão no jogo
- Verifique se o ped suporta tatuagens

### Menu não abre
- Verifique se o ox_lib está rodando
- Confirme que `ensure illenium-appearance` está no server.cfg
- Olhe o console do cliente (F8) para erros

### Roupas de emprego não funcionam
- Verifique se `Config.EnableJobClothing = true`
- Confirme que o job do jogador está correto
- Verifique a configuração em `Config.JobOutfits`

### Erro de framework não detectado
- Verifique se `Config.Framework` está definido corretamente
- Confirme que o framework está startado antes do illenium-appearance
- Verifique a pasta `shared/framework/` para suporte ao seu framework

### Conflito com qb-clothing
- O illenium-appearance substitui o qb-clothing
- Remova o qb-clothing do server.cfg
- Migre as skins existentes (processo automático)

---

## 📚 Links
- [Documentação](https://docs.illenium.dev/free-resources/illenium-appearance/installation/)
- [Discord](https://discord.illenium.dev)
- [GitHub](https://github.com/iLLeniumStudios/illenium-appearance)
- [Issue Tracker](https://github.com/iLLeniumStudios/illenium-appearance/issues)
