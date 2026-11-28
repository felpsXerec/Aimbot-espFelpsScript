-- BH Hub v5 | Belo Horizonte MG | Xeno
-- Key: ??? | LeftAlt abre/fecha o menu (configuravel)
-- v5: notificacoes com som, imagem fixa do dono, inventario real, otimizado
--     (veja BRAND_URL e OWNER_IDS no topo do script)

local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")
local Stats = game:GetService("Stats")

local LP = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local GuiParent = (gethui and gethui()) or CoreGui
local KEY = "HFDEGH-FJHKWA-32GHJD"

-- ====== CONFIGURACAO DO DONO ======
-- BRAND_URL: link RAW (https) de um arquivo de texto com UMA linha: o ID da imagem
--   (ex: 123456789), rbxassetid://... ou um link https de imagem. So quem controla esse
--   arquivo consegue trocar a imagem, e ela muda para TODOS os usuarios.
-- OWNER_IDS: UserIds do dono (libera o painel "Dono" no menu).
local BRAND_URL = ""
local OWNER_IDS = {7415387080}
-- ==================================

pcall(function()
    local old = GuiParent:FindFirstChild("BHHubV3")
    if old then old:Destroy() end
end)

--------------------------------------------------------------------
-- ESTADO
--------------------------------------------------------------------
local S = {
    ESP = false, Box = false, Health = false, Skeleton = false, Name = false,
    Distance = false, Weapon = false, Team = false, TeamCheck = false,
    Chams = false, Tracer = false, ESPVehicles = false, MaxDist = 1500,
    ColorESP = Color3.fromRGB(255, 255, 255),
    ColorVisible = Color3.fromRGB(0, 120, 215),
    ColorHidden = Color3.fromRGB(255, 0, 0),
    ColorTracer = Color3.fromRGB(0, 120, 215),
    ColorSkeleton = Color3.fromRGB(0, 120, 215),
    ColorChams = Color3.fromRGB(0, 120, 215),

    Aimbot = false, AimFOV = 150, AimSmooth = 0.25, AimPart = "Head",
    AimWall = true, AimTeam = false, ShowFOV = true,

    Hitbox = false, HitboxSize = 8, HitboxTransp = 0.6, HitboxTeam = false,

    InstantPrompts = false, RangeActive = false, PromptRange = 20, AutoInteract = false,
    Speed = false, SpeedValue = 40, InfJump = false, NoClip = false,
}

local KeyScreenOpen = true
local MenuKey = Enum.KeyCode.LeftAlt
pcall(function()
    if isfile and isfile("BHHub_menukey.txt") then
        local k = Enum.KeyCode[readfile("BHHub_menukey.txt")]
        if k then MenuKey = k end
    end
end)
local ListeningKey = false
local Connections = {}
local function Connect(sig, fn)
    local c = sig:Connect(fn)
    Connections[#Connections + 1] = c
    return c
end

local function Tween(o, t, props, style, dir)
    local tw = TweenService:Create(o, TweenInfo.new(t, style or Enum.EasingStyle.Quint, dir or Enum.EasingDirection.Out), props)
    tw:Play()
    return tw
end

-- ossos (usados no ESP real e no preview)
local R15 = {
    { "Head", "UpperTorso" }, { "UpperTorso", "LowerTorso" },
    { "UpperTorso", "LeftUpperArm" }, { "LeftUpperArm", "LeftLowerArm" }, { "LeftLowerArm", "LeftHand" },
    { "UpperTorso", "RightUpperArm" }, { "RightUpperArm", "RightLowerArm" }, { "RightLowerArm", "RightHand" },
    { "LowerTorso", "LeftUpperLeg" }, { "LeftUpperLeg", "LeftLowerLeg" }, { "LeftLowerLeg", "LeftFoot" },
    { "LowerTorso", "RightUpperLeg" }, { "RightUpperLeg", "RightLowerLeg" }, { "RightLowerLeg", "RightFoot" },
}
local R6 = {
    { "Head", "Torso" }, { "Torso", "Left Arm" }, { "Torso", "Right Arm" },
    { "Torso", "Left Leg" }, { "Torso", "Right Leg" },
}

--------------------------------------------------------------------
-- DRAWING (fallback)
--------------------------------------------------------------------
local Dummy = setmetatable({}, {
    __newindex = function() end,
    __index = function() return function() end end,
})
local function NewDraw(t, props)
    local ok, d = pcall(Drawing.new, t)
    if not ok or not d then return Dummy end
    -- alguns executores criam o desenho visivel em (0,0): esconde antes de tudo
    pcall(function() d.Visible = false end)
    for k, v in pairs(props or {}) do d[k] = v end
    return d
end

--------------------------------------------------------------------
-- TEMA / CORES
--------------------------------------------------------------------
-- menu todo preto: so a cor de destaque muda com o tema
local BG = Color3.fromRGB(5, 5, 5)
local BG2 = Color3.fromRGB(12, 12, 12)
local SIDE = Color3.fromRGB(0, 0, 0)
local ITEM = Color3.fromRGB(15, 15, 15)
local HOVER = Color3.fromRGB(30, 30, 30)
local OFF = Color3.fromRGB(42, 42, 42)
local STROKE = Color3.fromRGB(38, 38, 38)
local TEXT = Color3.fromRGB(228, 228, 232)
local MUTED = Color3.fromRGB(125, 125, 132)
local ACCENT = Color3.fromRGB(205, 205, 212)

local ThemeHooks = {}
local function OnTheme(fn) ThemeHooks[#ThemeHooks + 1] = fn end
local function ApplyTheme(c)
    ACCENT = c
    for _, fn in ipairs(ThemeHooks) do pcall(fn, c) end
end
-- cor do texto/ponto em cima da cor de destaque (preto se o destaque for claro)
local function OnAccent(c)
    return (c.R * 0.3 + c.G * 0.59 + c.B * 0.11) > 0.6 and Color3.new(0, 0, 0) or Color3.new(1, 1, 1)
end

local Themes = {
    { "Preto", Color3.fromRGB(205, 205, 212) },
    { "Vermelho", Color3.fromRGB(235, 50, 60) },
    { "Verde", Color3.fromRGB(0, 210, 110) },
    { "Roxo", Color3.fromRGB(150, 95, 255) },
}
local CurrentTheme = "Preto"
pcall(function()
    if isfile and isfile("BHHub_theme.txt") then
        local n = readfile("BHHub_theme.txt")
        for _, t in ipairs(Themes) do
            if t[1] == n then CurrentTheme = n; ACCENT = t[2] end
        end
    end
end)

--------------------------------------------------------------------
-- UI BASE
--------------------------------------------------------------------
local Gui = Instance.new("ScreenGui")
Gui.Name = "BHHubV3"
Gui.ResetOnSpawn = false
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.DisplayOrder = 999
Gui.Parent = GuiParent

local function Corner(o, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 6)
    c.Parent = o
    return c
end

local function Txt(parent, text, size, color, align)
    local l = Instance.new("TextLabel")
    l.BackgroundTransparency = 1
    l.Text = text
    l.Font = Enum.Font.GothamMedium
    l.TextSize = size or 13
    l.TextColor3 = color or TEXT
    l.TextXAlignment = align or Enum.TextXAlignment.Left
    l.Parent = parent
    return l
end

local RefreshPreview = function() end
local function Changed() RefreshPreview() end

local MainScale -- definido abaixo

local function Draggable(handle, target)
    local dragging, startPos, startInput
    Connect(handle.InputBegan, function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            startInput = i.Position
            startPos = target.Position
        end
    end)
    Connect(UserInputService.InputChanged, function(i)
        if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
            local d = i.Position - startInput
            target.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
    Connect(UserInputService.InputEnded, function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

-- slider: uma conexao so
local ActiveSlider = nil
Connect(UserInputService.InputChanged, function(i)
    if ActiveSlider and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        ActiveSlider(i.Position.X)
    end
end)
Connect(UserInputService.InputEnded, function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        ActiveSlider = nil
    end
end)

-- NOTIFICACOES + SONS ----------------------------------------------
-- Toast(titulo, tipo, descricao, chave)
--   tipo: "info" | "ok" | "off" | "warn" | "err"
--   chave: toasts com a mesma chave se substituem (evita spam)
local Toast, PlaySfx
local Sfx = { On = true, Last = 0 }
do
    local Holder = Instance.new("Frame")
    Holder.BackgroundTransparency = 1
    Holder.AnchorPoint = Vector2.new(1, 1)
    Holder.Position = UDim2.new(1, -18, 1, -18)
    Holder.Size = UDim2.new(0, 310, 1, -36)
    Holder.ZIndex = 50
    Holder.Parent = Gui
    local lay = Instance.new("UIListLayout")
    lay.Padding = UDim.new(0, 8)
    lay.VerticalAlignment = Enum.VerticalAlignment.Bottom
    lay.HorizontalAlignment = Enum.HorizontalAlignment.Right
    lay.SortOrder = Enum.SortOrder.LayoutOrder
    lay.Parent = Holder

    local Kinds = {
        info = { Color3.fromRGB(120, 170, 255), "i" },
        ok = { Color3.fromRGB(70, 220, 130), "+" },
        off = { Color3.fromRGB(150, 150, 160), "-" },
        warn = { Color3.fromRGB(255, 190, 60), "!" },
        err = { Color3.fromRGB(255, 85, 85), "x" },
    }

    -- sons nativos do cliente do Roblox (nao dependem de upload). Um objeto por tipo;
    -- antes de tocar um, o anterior e parado: nunca ha dois sons ao mesmo tempo.
    local SoundFiles = {
        ok = "rbxasset://sounds/electronicpingshort.wav",
        off = "rbxasset://sounds/switch.wav",
        info = "rbxasset://sounds/button.wav",
        warn = "rbxasset://sounds/button.wav",
        err = "rbxasset://sounds/snap.mp3",
    }
    local Sounds, Playing = {}, nil
    for k, id in pairs(SoundFiles) do
        local snd = Instance.new("Sound")
        snd.SoundId = id
        snd.Volume = 0.25
        snd.Parent = Gui
        Sounds[k] = snd
    end
    PlaySfx = function(kind)
        if not Sfx.On then return end
        local now = os.clock()
        if now - (Sfx.LastSound or 0) < 0.12 then return end
        Sfx.LastSound = now
        local snd = Sounds[kind] or Sounds.info
        pcall(function()
            if Playing and Playing ~= snd then Playing:Stop() end
            Playing = snd
            snd.TimePosition = 0
            snd:Play()
        end)
    end

    local N = 0
    local Keyed = {}
    Toast = function(title, kind, desc, key)
        if not Gui.Parent then return end
        local def = Kinds[kind] or Kinds.info
        kind = Kinds[kind] and kind or "info"
        local col = def[1]
        Sfx.Last = os.clock()

        if key and Keyed[key] and Keyed[key].Parent then Keyed[key]:Destroy() end
        local alive = {}
        for _, c in ipairs(Holder:GetChildren()) do
            if c:IsA("Frame") then alive[#alive + 1] = c end
        end
        if #alive >= 4 then alive[1]:Destroy() end

        N = N + 1
        local wrap = Instance.new("Frame")
        wrap.BackgroundTransparency = 1
        wrap.Size = UDim2.new(1, 0, 0, 54)
        wrap.LayoutOrder = N
        wrap.Parent = Holder
        if key then Keyed[key] = wrap end

        local t = Instance.new("CanvasGroup")
        t.Size = UDim2.new(1, 0, 1, 0)
        t.Position = UDim2.new(0, 330, 0, 0)
        t.BackgroundColor3 = BG2
        t.GroupTransparency = 1
        t.BorderSizePixel = 0
        t.Parent = wrap
        Corner(t, 10)
        local st = Instance.new("UIStroke")
        st.Color = STROKE
        st.Thickness = 1
        st.Parent = t

        -- icone
        local ic = Instance.new("Frame")
        ic.BackgroundColor3 = col
        ic.BackgroundTransparency = 0.8
        ic.BorderSizePixel = 0
        ic.Position = UDim2.new(0, 12, 0.5, -14)
        ic.Size = UDim2.new(0, 28, 0, 28)
        ic.Parent = t
        Corner(ic, 14)
        local il = Txt(ic, def[2], 16, col, Enum.TextXAlignment.Center)
        il.Font = Enum.Font.GothamBlack
        il.Size = UDim2.new(1, 0, 1, 0)

        local hasDesc = desc and desc ~= ""
        local tl = Txt(t, title, 13, Color3.new(1, 1, 1))
        tl.Font = Enum.Font.GothamBold
        tl.Position = UDim2.new(0, 52, 0, hasDesc and 9 or 17)
        tl.Size = UDim2.new(1, -60, 0, 18)
        tl.TextTruncate = Enum.TextTruncate.AtEnd
        if hasDesc then
            local dl = Txt(t, desc, 11, MUTED)
            dl.Position = UDim2.new(0, 52, 0, 27)
            dl.Size = UDim2.new(1, -60, 0, 16)
            dl.TextTruncate = Enum.TextTruncate.AtEnd
        end
        local prog = Instance.new("Frame")
        prog.BackgroundColor3 = col
        prog.BorderSizePixel = 0
        prog.AnchorPoint = Vector2.new(0, 1)
        prog.Position = UDim2.new(0, 0, 1, 0)
        prog.Size = UDim2.new(1, 0, 0, 2)
        prog.Parent = t

        PlaySfx(kind)
        Tween(t, 0.4, { Position = UDim2.new(0, 0, 0, 0), GroupTransparency = 0 }, Enum.EasingStyle.Back)
        Tween(prog, 2.6, { Size = UDim2.new(0, 0, 0, 2) }, Enum.EasingStyle.Linear)
        task.delay(2.7, function()
            if not wrap.Parent then return end
            Tween(t, 0.3, { Position = UDim2.new(0, 330, 0, 0), GroupTransparency = 1 }, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
            task.delay(0.3, function()
                if not wrap.Parent then return end
                Tween(wrap, 0.2, { Size = UDim2.new(1, 0, 0, 0) })
                task.delay(0.22, function()
                    if Keyed[key or 0] == wrap then Keyed[key] = nil end
                    wrap:Destroy()
                end)
            end)
        end)
    end
end

-- spinner de carregamento (anel girando)
local function MakeSpinner(parent, size, thick)
    local f = Instance.new("Frame")
    f.BackgroundTransparency = 1
    f.Size = UDim2.new(0, size, 0, size)
    f.Parent = parent
    Corner(f, size / 2)
    local st = Instance.new("UIStroke")
    st.Color = ACCENT
    st.Thickness = thick or 2.5
    st.Parent = f
    local g = Instance.new("UIGradient")
    g.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.55, 1),
        NumberSequenceKeypoint.new(1, 1),
    })
    g.Parent = st
    local tw = TweenService:Create(g, TweenInfo.new(0.9, Enum.EasingStyle.Linear, Enum.EasingDirection.Out, -1), { Rotation = 360 })
    tw:Play()
    OnTheme(function(c) st.Color = c end)
    return f, tw
end

--------------------------------------------------------------------
-- JANELA PRINCIPAL
--------------------------------------------------------------------
local MAIN_W, MAIN_H = 840, 440

local Main = Instance.new("CanvasGroup")
Main.AnchorPoint = Vector2.new(0.5, 0.5)
Main.Size = UDim2.new(0, MAIN_W, 0, MAIN_H)
Main.Position = UDim2.new(0.5, 0, 0.5, 0)
Main.BackgroundColor3 = BG
Main.BorderSizePixel = 0
Main.Visible = false
Main.GroupTransparency = 1
Main.Parent = Gui
Corner(Main, 10)
MainScale = Instance.new("UIScale")
MainScale.Scale = 0.88
MainScale.Parent = Main

-- escala automatica: o menu encolhe em telas pequenas
local function BaseScale()
    local v = (Workspace.CurrentCamera or Camera).ViewportSize
    return math.clamp(math.min((v.X - 40) / MAIN_W, (v.Y - 40) / MAIN_H), 0.5, 1)
end

-- sombra suave atras do menu (camadas translucidas)
local Shadow = Instance.new("CanvasGroup")
Shadow.AnchorPoint = Vector2.new(0.5, 0.5)
Shadow.BackgroundTransparency = 1
Shadow.Size = UDim2.new(0, MAIN_W + 100, 0, MAIN_H + 100)
Shadow.Position = Main.Position
Shadow.GroupTransparency = 1
Shadow.Visible = false
Shadow.ZIndex = 0
Shadow.Parent = Gui
local ShadowScale = Instance.new("UIScale")
ShadowScale.Parent = Shadow
for _, g in ipairs({ 6, 14, 24, 36 }) do
    local l = Instance.new("Frame")
    l.AnchorPoint = Vector2.new(0.5, 0.5)
    l.Position = UDim2.new(0.5, 0, 0.5, 10)
    l.Size = UDim2.new(0, MAIN_W + g * 2, 0, MAIN_H + g * 2)
    l.BackgroundColor3 = Color3.new(0, 0, 0)
    l.BackgroundTransparency = 0.93
    l.BorderSizePixel = 0
    l.Parent = Shadow
    Corner(l, 10 + g)
end

-- desfoque do jogo atras do menu (opcional, em Configuracoes)
local Blur = Instance.new("BlurEffect")
Blur.Size = 0
local BlurOn = false
pcall(function() Blur.Parent = Lighting end)

-- o gradiente fica num frame de fundo: UIGradient direto no CanvasGroup escurece tudo
local MainBG = Instance.new("Frame")
MainBG.Size = UDim2.new(1, 0, 1, 0)
MainBG.BackgroundColor3 = Color3.new(1, 1, 1)
MainBG.BorderSizePixel = 0
MainBG.ZIndex = 0
MainBG.Parent = Main
local MainGrad = Instance.new("UIGradient")
MainGrad.Color = ColorSequence.new(BG, BG2)
MainGrad.Rotation = 45
MainGrad.Parent = MainBG

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = ACCENT
MainStroke.Thickness = 1.5
MainStroke.Transparency = 0.5
MainStroke.Parent = Main
OnTheme(function(c) MainStroke.Color = c end)

local Side = Instance.new("Frame")
Side.Size = UDim2.new(0, 150, 1, 0)
Side.BackgroundColor3 = SIDE
Side.BorderSizePixel = 0
Side.Parent = Main

-- icone do menu (canto superior esquerdo): troque em Configuracoes > Icone do menu
local IconBtn = Instance.new("TextButton")
IconBtn.Position = UDim2.new(0, 8, 0, 8)
IconBtn.Size = UDim2.new(1, -16, 0, 64)
IconBtn.BackgroundColor3 = ITEM
IconBtn.BorderSizePixel = 0
IconBtn.Text = ""
IconBtn.AutoButtonColor = false
IconBtn.ClipsDescendants = true
IconBtn.Parent = Side
Corner(IconBtn, 8)
local IconStroke = Instance.new("UIStroke")
IconStroke.Color = ACCENT
IconStroke.Thickness = 1.5
IconStroke.Transparency = 0.3
IconStroke.Parent = IconBtn
OnTheme(function(c) IconStroke.Color = c end)

local IconFallback = Txt(IconBtn, "BH", 15, ACCENT, Enum.TextXAlignment.Center)
IconFallback.Font = Enum.Font.GothamBlack
IconFallback.Size = UDim2.new(1, 0, 1, 0)
OnTheme(function(c) IconFallback.TextColor3 = c end)

local IconImg = Instance.new("ImageLabel")
IconImg.BackgroundTransparency = 1
IconImg.Size = UDim2.new(1, 0, 1, 0)
IconImg.ScaleType = Enum.ScaleType.Crop
IconImg.Visible = false
IconImg.Parent = IconBtn
Corner(IconImg, 8)

IconBtn.MouseEnter:Connect(function() Tween(IconBtn, 0.2, { BackgroundColor3 = HOVER }) end)
IconBtn.MouseLeave:Connect(function() Tween(IconBtn, 0.2, { BackgroundColor3 = ITEM }) end)

local Logo = Txt(Side, "BH HUB", 20, Color3.new(1, 1, 1), Enum.TextXAlignment.Left)
Logo.Font = Enum.Font.GothamBlack
Logo.Position = UDim2.new(0, 52, 0, 0)
Logo.Size = UDim2.new(1, -56, 0, 52)
Logo.Visible = false -- o banner ja traz o nome
local LogoGrad = Instance.new("UIGradient")
LogoGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, ACCENT),
    ColorSequenceKeypoint.new(0.5, Color3.new(1, 1, 1)),
    ColorSequenceKeypoint.new(1, ACCENT),
})
LogoGrad.Parent = Logo
OnTheme(function(c)
    LogoGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, c),
        ColorSequenceKeypoint.new(0.5, Color3.new(1, 1, 1)),
        ColorSequenceKeypoint.new(1, c),
    })
end)

local TabHolder = Instance.new("Frame")
TabHolder.BackgroundTransparency = 1
TabHolder.Position = UDim2.new(0, 8, 0, 80)
TabHolder.Size = UDim2.new(1, -16, 1, -140)
TabHolder.Parent = Side
local tabLayout = Instance.new("UIListLayout")
tabLayout.Padding = UDim.new(0, 4)
tabLayout.Parent = TabHolder

local Indicator = Instance.new("Frame")
Indicator.Size = UDim2.new(0, 3, 0, 20)
Indicator.Position = UDim2.new(0, 0, 0, 86)
Indicator.BackgroundColor3 = ACCENT
Indicator.BorderSizePixel = 0
Indicator.Parent = Side
Corner(Indicator, 2)
OnTheme(function(c) Tween(Indicator, 0.3, { BackgroundColor3 = c }) end)

-- rodape com a foto do usuario
local Footer = Instance.new("Frame")
Footer.BackgroundColor3 = BG2
Footer.BorderSizePixel = 0
Footer.Position = UDim2.new(0, 8, 1, -52)
Footer.Size = UDim2.new(1, -16, 0, 44)
Footer.Parent = Side
Corner(Footer, 8)

local Avatar = Instance.new("ImageLabel")
Avatar.BackgroundColor3 = OFF
Avatar.BorderSizePixel = 0
Avatar.Position = UDim2.new(0, 7, 0.5, -15)
Avatar.Size = UDim2.new(0, 30, 0, 30)
Avatar.Parent = Footer
Corner(Avatar, 15)
task.spawn(function()
    local ok, img = pcall(function()
        return Players:GetUserThumbnailAsync(LP.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size100x100)
    end)
    if ok and Avatar.Parent then Avatar.Image = img end
end)

local UserLbl = Txt(Footer, LP.DisplayName, 12, TEXT)
UserLbl.Font = Enum.Font.GothamBold
UserLbl.Position = UDim2.new(0, 44, 0, 6)
UserLbl.Size = UDim2.new(1, -50, 0, 16)
UserLbl.TextTruncate = Enum.TextTruncate.AtEnd
local UserSub = Txt(Footer, "Cliente", 11, MUTED)
UserSub.Position = UDim2.new(0, 44, 0, 22)
UserSub.Size = UDim2.new(1, -50, 0, 14)

local HintLbl = Txt(Side, "", 10, MUTED, Enum.TextXAlignment.Center)
HintLbl.Position = UDim2.new(0, 0, 1, -68)
HintLbl.Size = UDim2.new(1, 0, 0, 14)

local TopBar = Instance.new("Frame")
TopBar.BackgroundTransparency = 1
TopBar.Position = UDim2.new(0, 150, 0, 0)
TopBar.Size = UDim2.new(1, -150, 0, 40)
TopBar.Parent = Main
Draggable(TopBar, Main)
Draggable(Side, Main)

local TitleLbl = Txt(TopBar, "Visuais", 15, TEXT)
TitleLbl.Position = UDim2.new(0, 16, 0, 0)
TitleLbl.Size = UDim2.new(0, 200, 1, 0)
TitleLbl.Font = Enum.Font.GothamBold

local StatsLbl = Txt(TopBar, "-- FPS | --ms", 11, MUTED, Enum.TextXAlignment.Right)
StatsLbl.Position = UDim2.new(1, -190, 0, 0)
StatsLbl.Size = UDim2.new(0, 140, 1, 0)

local MenuOpen = false
local IntroDone = false
local OnIntroDone -- definido junto das paginas

-- tela de carregamento (so na primeira abertura)
local function RunIntro()
    local ov = Instance.new("CanvasGroup")
    ov.Size = UDim2.new(1, 0, 1, 0)
    ov.BackgroundColor3 = SIDE
    ov.BorderSizePixel = 0
    ov.Active = true
    ov.ZIndex = 100
    ov.Parent = Main
    local lg = Txt(ov, "BH HUB", 26, Color3.new(1, 1, 1), Enum.TextXAlignment.Center)
    lg.Font = Enum.Font.GothamBlack
    lg.Position = UDim2.new(0, 0, 0.5, -64)
    lg.Size = UDim2.new(1, 0, 0, 34)
    local sp, tw = MakeSpinner(ov, 30, 3)
    sp.AnchorPoint = Vector2.new(0.5, 0)
    sp.Position = UDim2.new(0.5, 0, 0.5, -22)
    local status = Txt(ov, "", 12, MUTED, Enum.TextXAlignment.Center)
    status.Position = UDim2.new(0, 0, 0.5, 20)
    status.Size = UDim2.new(1, 0, 0, 16)
    local track = Instance.new("Frame")
    track.AnchorPoint = Vector2.new(0.5, 0)
    track.Position = UDim2.new(0.5, 0, 0.5, 46)
    track.Size = UDim2.new(0, 220, 0, 4)
    track.BackgroundColor3 = OFF
    track.BorderSizePixel = 0
    track.Parent = ov
    Corner(track, 2)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BackgroundColor3 = ACCENT
    fill.BorderSizePixel = 0
    fill.Parent = track
    Corner(fill, 2)
    local steps = { "Carregando modulos...", "Lendo inventario...", "Aplicando tema...", "Pronto!" }
    for i, s in ipairs(steps) do
        if not ov.Parent then return end
        status.Text = s
        Tween(fill, 0.22, { Size = UDim2.new(i / #steps, 0, 1, 0) })
        task.wait(0.24)
    end
    tw:Cancel()
    Tween(ov, 0.4, { GroupTransparency = 1 })
    task.delay(0.42, function() ov:Destroy() end)
    if OnIntroDone then OnIntroDone() end
end

local function SetMenu(v)
    MenuOpen = v
    if v then
        Main.Visible = true
        Shadow.Visible = true
        Tween(Main, 0.35, { GroupTransparency = 0 })
        Tween(Shadow, 0.5, { GroupTransparency = 0 })
        Tween(MainScale, 0.45, { Scale = BaseScale() }, Enum.EasingStyle.Back)
        if BlurOn then Tween(Blur, 0.3, { Size = 12 }) end
        if not IntroDone then
            IntroDone = true
            task.spawn(RunIntro)
        end
    else
        Tween(Main, 0.2, { GroupTransparency = 1 })
        Tween(Shadow, 0.2, { GroupTransparency = 1 })
        Tween(MainScale, 0.2, { Scale = BaseScale() * 0.92 })
        Tween(Blur, 0.2, { Size = 0 })
        task.delay(0.22, function()
            if not MenuOpen then
                Main.Visible = false
                Shadow.Visible = false
            end
        end)
    end
end

local CloseBtn = Instance.new("TextButton")
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "X"
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 14
CloseBtn.TextColor3 = MUTED
CloseBtn.Position = UDim2.new(1, -36, 0, 0)
CloseBtn.Size = UDim2.new(0, 30, 1, 0)
CloseBtn.Parent = TopBar
CloseBtn.MouseEnter:Connect(function() Tween(CloseBtn, 0.15, { TextColor3 = Color3.fromRGB(255, 90, 90) }) end)
CloseBtn.MouseLeave:Connect(function() Tween(CloseBtn, 0.15, { TextColor3 = MUTED }) end)
CloseBtn.MouseButton1Click:Connect(function() SetMenu(false) end)

--------------------------------------------------------------------
-- PAGINAS, ABAS E ANIMACAO DOS ITENS
--------------------------------------------------------------------
local Pages, TabButtons, TabIndex, PageItems = {}, {}, {}, {}
local TabCount = 0
local Current = nil
local PreviewActive = false
local PreviewPanel

local PAGE_W_FULL = MAIN_W - 172
local PAGE_W_PREV = 410

local function AnimatePage(scroll)
    local list = PageItems[scroll]
    if not list then return end
    for i, w in ipairs(list) do
        local inner = w:FindFirstChildWhichIsA("GuiObject")
        if inner then
            inner.Position = UDim2.new(0, 34, 0, 0)
            task.delay(math.min(i * 0.03, 0.6), function()
                if inner.Parent then
                    Tween(inner, 0.45, { Position = UDim2.new(0, 0, 0, 0) }, Enum.EasingStyle.Back)
                end
            end)
        end
    end
end

OnIntroDone = function()
    local h = Current and Pages[Current]
    local sc = h and h:FindFirstChildWhichIsA("ScrollingFrame")
    if sc then AnimatePage(sc) end
end

local OnInvShown -- definido na aba Inventario
local TabChips = {}

local function ShowPage(name, instant)
    if Current == name then return end
    local old = Current
    Current = name
    if old and Pages[old] then Pages[old].Visible = false end

    local holder = Pages[name]
    local hasPrev = (name == "Visuais")
    holder.Size = UDim2.new(0, hasPrev and PAGE_W_PREV or PAGE_W_FULL, 1, -54)
    holder.Visible = true
    if instant then
        holder.GroupTransparency = 0
        holder.Position = UDim2.new(0, 160, 0, 44)
    else
        holder.GroupTransparency = 1
        holder.Position = UDim2.new(0, 160, 0, 58)
        Tween(holder, 0.35, { GroupTransparency = 0, Position = UDim2.new(0, 160, 0, 44) })
    end
    AnimatePage(holder:FindFirstChildWhichIsA("ScrollingFrame"))

    PreviewActive = hasPrev
    if PreviewPanel then
        if hasPrev then
            PreviewPanel.Visible = true
            PreviewPanel.GroupTransparency = 1
            PreviewPanel.Position = UDim2.new(0, MAIN_W - 252 + 30, 0, 44)
            Tween(PreviewPanel, 0.5, { GroupTransparency = 0, Position = UDim2.new(0, MAIN_W - 252, 0, 44) }, Enum.EasingStyle.Back)
        else
            Tween(PreviewPanel, 0.2, { GroupTransparency = 1 })
            task.delay(0.22, function()
                if not PreviewActive then PreviewPanel.Visible = false end
            end)
        end
    end

    for n, b in pairs(TabButtons) do
        local sel = (n == name)
        Tween(b, 0.2, { BackgroundColor3 = sel and ITEM or SIDE, TextColor3 = sel and ACCENT or MUTED })
        local chip = TabChips[n]
        if chip then
            Tween(chip.Frame, 0.25, { BackgroundColor3 = sel and ACCENT or OFF })
            Tween(chip.Label, 0.25, { TextColor3 = sel and OnAccent(ACCENT) or MUTED })
            if sel then
                chip.Frame.Size = UDim2.new(0, 14, 0, 14)
                Tween(chip.Frame, 0.4, { Size = UDim2.new(0, 22, 0, 22) }, Enum.EasingStyle.Back)
            end
        end
    end
    if name == "Inventario" and OnInvShown then task.spawn(OnInvShown) end
    if not instant then Toast("Aba " .. name, "info", "Abrindo " .. name, "tab") end
    Tween(Indicator, 0.35, { Position = UDim2.new(0, 0, 0, 80 + (TabIndex[name] - 1) * 36 + 6) }, Enum.EasingStyle.Back)

    TitleLbl.Text = name
    TitleLbl.TextTransparency = 1
    TitleLbl.Position = UDim2.new(0, 28, 0, 0)
    Tween(TitleLbl, 0.35, { TextTransparency = 0, Position = UDim2.new(0, 16, 0, 0) })
end
OnTheme(function(c)
    if Current and TabButtons[Current] then TabButtons[Current].TextColor3 = c end
    local chip = Current and TabChips[Current]
    if chip then
        chip.Frame.BackgroundColor3 = c
        chip.Label.TextColor3 = OnAccent(c)
    end
end)

local function AddTab(name)
    TabCount = TabCount + 1
    TabIndex[name] = TabCount

    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 32)
    b.BackgroundColor3 = SIDE
    b.BorderSizePixel = 0
    b.Text = name
    b.TextXAlignment = Enum.TextXAlignment.Left
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 13
    b.TextColor3 = MUTED
    b.AutoButtonColor = false
    b.Parent = TabHolder
    Corner(b, 6)
    local pad = Instance.new("UIPadding")
    pad.PaddingLeft = UDim.new(0, 40)
    pad.Parent = b

    -- icone da aba (chip com a inicial)
    local chipF = Instance.new("Frame")
    chipF.AnchorPoint = Vector2.new(0.5, 0.5)
    chipF.Position = UDim2.new(0, -21, 0.5, 0)
    chipF.Size = UDim2.new(0, 22, 0, 22)
    chipF.BackgroundColor3 = OFF
    chipF.BorderSizePixel = 0
    chipF.Parent = b
    Corner(chipF, 6)
    local chipL = Txt(chipF, string.upper(string.sub(name, 1, 1)), 12, MUTED, Enum.TextXAlignment.Center)
    chipL.Font = Enum.Font.GothamBlack
    chipL.Size = UDim2.new(1, 0, 1, 0)
    TabChips[name] = { Frame = chipF, Label = chipL }

    local holder = Instance.new("CanvasGroup")
    holder.BackgroundTransparency = 1
    holder.Position = UDim2.new(0, 160, 0, 44)
    holder.Size = UDim2.new(0, PAGE_W_FULL, 1, -54)
    holder.Visible = false
    holder.Parent = Main

    local page = Instance.new("ScrollingFrame")
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.Size = UDim2.new(1, 0, 1, 0)
    page.CanvasSize = UDim2.new(0, 0, 0, 0)
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.ScrollBarThickness = 3
    page.ScrollBarImageColor3 = ACCENT
    page.Parent = holder
    OnTheme(function(c) page.ScrollBarImageColor3 = c end)
    local lay = Instance.new("UIListLayout")
    lay.Padding = UDim.new(0, 6)
    lay.Parent = page

    Pages[name] = holder
    TabButtons[name] = b

    b.MouseEnter:Connect(function()
        if Current ~= name then Tween(b, 0.15, { BackgroundColor3 = HOVER, TextColor3 = TEXT }) end
    end)
    b.MouseLeave:Connect(function()
        if Current ~= name then Tween(b, 0.15, { BackgroundColor3 = SIDE, TextColor3 = MUTED }) end
    end)
    b.MouseButton1Click:Connect(function() ShowPage(name) end)
    return page
end

--------------------------------------------------------------------
-- COMPONENTES (com wrapper para animar entrada)
--------------------------------------------------------------------
local function NewItem(page, h)
    local w = Instance.new("Frame")
    w.BackgroundTransparency = 1
    w.Size = UDim2.new(1, -8, 0, h)
    w.Parent = page
    PageItems[page] = PageItems[page] or {}
    table.insert(PageItems[page], w)
    return w
end

local function Ripple(btn)
    btn.ClipsDescendants = true
    btn.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            local rel = (Vector2.new(i.Position.X, i.Position.Y) - btn.AbsolutePosition) / math.max(MainScale.Scale, 0.1)
            local c = Instance.new("Frame")
            c.AnchorPoint = Vector2.new(0.5, 0.5)
            c.Position = UDim2.new(0, rel.X, 0, rel.Y)
            c.Size = UDim2.new(0, 0, 0, 0)
            c.BackgroundColor3 = ACCENT
            c.BackgroundTransparency = 0.7
            c.BorderSizePixel = 0
            c.ZIndex = 0
            c.Parent = btn
            Corner(c, 100)
            local d = math.max(btn.AbsoluteSize.X, 60) * 1.6 / math.max(MainScale.Scale, 0.1)
            Tween(c, 0.5, { Size = UDim2.new(0, d, 0, d), BackgroundTransparency = 1 })
            task.delay(0.55, function() c:Destroy() end)
        end
    end)
end

local function Style(obj, withRipple)
    Corner(obj, 7)
    local st = Instance.new("UIStroke")
    st.Color = STROKE
    st.Thickness = 1
    st.Transparency = 0.35
    st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    st.Parent = obj
    obj.MouseEnter:Connect(function()
        Tween(obj, 0.18, { BackgroundColor3 = HOVER })
        Tween(st, 0.18, { Color = ACCENT, Transparency = 0.15 })
    end)
    obj.MouseLeave:Connect(function()
        Tween(obj, 0.18, { BackgroundColor3 = ITEM })
        Tween(st, 0.18, { Color = STROKE, Transparency = 0.35 })
    end)
    if withRipple then Ripple(obj) end
    return st
end

local function Section(page, text)
    local w = NewItem(page, 22)
    local l = Txt(w, text, 12, ACCENT)
    l.Font = Enum.Font.GothamBold
    l.Size = UDim2.new(1, 0, 1, 0)
    OnTheme(function(c) l.TextColor3 = c end)
end

local function AddToggle(page, text, default, cb)
    local w = NewItem(page, 32)
    local f = Instance.new("TextButton")
    f.Size = UDim2.new(1, 0, 1, 0)
    f.BackgroundColor3 = ITEM
    f.BorderSizePixel = 0
    f.Text = ""
    f.AutoButtonColor = false
    f.Parent = w
    Style(f, true)
    local l = Txt(f, text, 13, TEXT)
    l.Position = UDim2.new(0, 12, 0, 0)
    l.Size = UDim2.new(1, -50, 1, 0)
    local box = Instance.new("Frame")
    box.Size = UDim2.new(0, 18, 0, 18)
    box.Position = UDim2.new(1, -30, 0.5, -9)
    box.BackgroundColor3 = default and ACCENT or OFF
    box.BorderSizePixel = 0
    box.Parent = f
    Corner(box, 5)
    local dot = Instance.new("Frame")
    dot.AnchorPoint = Vector2.new(0.5, 0.5)
    dot.Position = UDim2.new(0.5, 0, 0.5, 0)
    dot.Size = default and UDim2.new(0, 8, 0, 8) or UDim2.new(0, 0, 0, 0)
    dot.BackgroundColor3 = OnAccent(ACCENT)
    dot.BorderSizePixel = 0
    dot.Parent = box
    Corner(dot, 3)

    local state = default
    OnTheme(function(c)
        dot.BackgroundColor3 = OnAccent(c)
        Tween(box, 0.25, { BackgroundColor3 = state and c or OFF })
    end)
    f.MouseEnter:Connect(function() Tween(l, 0.15, { Position = UDim2.new(0, 18, 0, 0) }) end)
    f.MouseLeave:Connect(function() Tween(l, 0.15, { Position = UDim2.new(0, 12, 0, 0) }) end)
    f.MouseButton1Click:Connect(function()
        state = not state
        Tween(box, 0.2, { BackgroundColor3 = state and ACCENT or OFF })
        Tween(dot, 0.25, { Size = state and UDim2.new(0, 8, 0, 8) or UDim2.new(0, 0, 0, 0) }, Enum.EasingStyle.Back)
        Tween(l, 0.2, { TextColor3 = state and Color3.new(1, 1, 1) or TEXT })
        cb(state)
        Changed()
        Toast(text, state and "ok" or "off", state and "Opcao ativada" or "Opcao desativada", "tg" .. text)
    end)
    if default then cb(true) end
end

local function AddSlider(page, text, min, max, default, cb, step)
    step = step or 1
    local w = NewItem(page, 46)
    local f = Instance.new("Frame")
    f.Size = UDim2.new(1, 0, 1, 0)
    f.BackgroundColor3 = ITEM
    f.BorderSizePixel = 0
    f.Parent = w
    Style(f, false)
    local l = Txt(f, text, 13, TEXT)
    l.Position = UDim2.new(0, 12, 0, 4)
    l.Size = UDim2.new(1, -24, 0, 18)
    local v = Txt(f, tostring(default), 13, MUTED, Enum.TextXAlignment.Right)
    v.Position = UDim2.new(0, 12, 0, 4)
    v.Size = UDim2.new(1, -24, 0, 18)
    local bar = Instance.new("Frame")
    bar.Position = UDim2.new(0, 12, 0, 30)
    bar.Size = UDim2.new(1, -24, 0, 6)
    bar.BackgroundColor3 = OFF
    bar.BorderSizePixel = 0
    bar.Parent = f
    Corner(bar, 3)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    fill.BackgroundColor3 = ACCENT
    fill.BorderSizePixel = 0
    fill.Parent = bar
    Corner(fill, 3)
    local knob = Instance.new("Frame")
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.Position = UDim2.new(1, 0, 0.5, 0)
    knob.Size = UDim2.new(0, 12, 0, 12)
    knob.BackgroundColor3 = Color3.new(1, 1, 1)
    knob.BorderSizePixel = 0
    knob.ZIndex = 2
    knob.Parent = fill
    Corner(knob, 6)
    OnTheme(function(c) fill.BackgroundColor3 = c end)

    local function setFromX(x)
        local rel = math.clamp((x - bar.AbsolutePosition.X) / math.max(bar.AbsoluteSize.X * MainScale.Scale, 1), 0, 1)
        local val = min + (max - min) * rel
        val = math.clamp(math.floor(val / step + 0.5) * step, min, max)
        Tween(fill, 0.08, { Size = UDim2.new((val - min) / (max - min), 0, 1, 0) }, Enum.EasingStyle.Linear)
        v.Text = tostring(math.floor(val * 100 + 0.5) / 100)
        cb(val)
        Changed()
    end
    bar.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            ActiveSlider = setFromX
            Tween(knob, 0.15, { Size = UDim2.new(0, 16, 0, 16) }, Enum.EasingStyle.Back)
            setFromX(i.Position.X)
        end
    end)
    Connect(UserInputService.InputEnded, function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            Tween(knob, 0.15, { Size = UDim2.new(0, 12, 0, 12) })
        end
    end)
end

local function AddButton(page, text, cb)
    local w = NewItem(page, 32)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 1, 0)
    b.BackgroundColor3 = ITEM
    b.BorderSizePixel = 0
    b.Text = text
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 13
    b.TextColor3 = TEXT
    b.AutoButtonColor = false
    b.Parent = w
    Style(b, true)
    b.MouseEnter:Connect(function() Tween(b, 0.15, { TextSize = 14, TextColor3 = Color3.new(1, 1, 1) }) end)
    b.MouseLeave:Connect(function() Tween(b, 0.15, { TextSize = 13, TextColor3 = TEXT }) end)
    b.MouseButton1Down:Connect(function() Tween(b, 0.08, { TextSize = 12 }) end)
    b.MouseButton1Up:Connect(function() Tween(b, 0.12, { TextSize = 14 }, Enum.EasingStyle.Back) end)
    b.MouseButton1Click:Connect(function()
        cb(b)
        -- aviso generico, se o proprio botao nao mostrou nenhum
        if text ~= "" then
            local t0 = os.clock()
            task.delay(0.05, function()
                if Sfx.Last < t0 then Toast(text, "info", "Funcao utilizada", "bt" .. text) end
            end)
        end
    end)
    return b
end

local function AddCycle(page, text, options, default, cb)
    local idx = table.find(options, default) or 1
    local b = AddButton(page, text .. ": " .. tostring(options[idx]), function() end)
    b.MouseButton1Click:Connect(function()
        idx = idx % #options + 1
        b.Text = text .. ": " .. tostring(options[idx])
        b.TextColor3 = ACCENT
        Tween(b, 0.35, { TextColor3 = Color3.new(1, 1, 1) })
        cb(options[idx])
        Changed()
        Toast(text, "info", tostring(options[idx]), "cy" .. text)
    end)
end

local ColorPresets = {
    Color3.fromRGB(255, 255, 255), Color3.fromRGB(0, 120, 215), Color3.fromRGB(255, 0, 0),
    Color3.fromRGB(0, 220, 90), Color3.fromRGB(255, 220, 0), Color3.fromRGB(170, 70, 255),
}
local function AddColor(page, text, key)
    local w = NewItem(page, 32)
    local f = Instance.new("TextButton")
    f.Size = UDim2.new(1, 0, 1, 0)
    f.BackgroundColor3 = ITEM
    f.BorderSizePixel = 0
    f.Text = ""
    f.AutoButtonColor = false
    f.Parent = w
    Style(f, true)
    local l = Txt(f, text, 13, TEXT)
    l.Position = UDim2.new(0, 12, 0, 0)
    l.Size = UDim2.new(1, -50, 1, 0)
    local sw = Instance.new("Frame")
    sw.Size = UDim2.new(0, 18, 0, 18)
    sw.Position = UDim2.new(1, -30, 0.5, -9)
    sw.BackgroundColor3 = S[key]
    sw.BorderSizePixel = 0
    sw.Parent = f
    Corner(sw, 9)
    local i = 0
    for n, c in ipairs(ColorPresets) do if c == S[key] then i = n end end
    f.MouseButton1Click:Connect(function()
        i = i % #ColorPresets + 1
        S[key] = ColorPresets[i]
        Tween(sw, 0.25, { BackgroundColor3 = S[key] })
        Tween(sw, 0.12, { Size = UDim2.new(0, 22, 0, 22), Position = UDim2.new(1, -32, 0.5, -11) })
        task.delay(0.12, function()
            Tween(sw, 0.2, { Size = UDim2.new(0, 18, 0, 18), Position = UDim2.new(1, -30, 0.5, -9) }, Enum.EasingStyle.Back)
        end)
        Changed()
    end)
end

--------------------------------------------------------------------
-- PREVIEW DO ESP COM A SKIN DO JOGADOR (ViewportFrame)
--------------------------------------------------------------------
local PV = {}
local PVVisible = true
local STAGE_W, STAGE_H = 240, MAIN_H - 54 - 36

do
    PreviewPanel = Instance.new("CanvasGroup")
    PreviewPanel.Size = UDim2.new(0, 240, 0, MAIN_H - 54)
    PreviewPanel.Position = UDim2.new(0, MAIN_W - 252, 0, 44)
    PreviewPanel.BackgroundColor3 = ITEM
    PreviewPanel.BorderSizePixel = 0
    PreviewPanel.Visible = false
    PreviewPanel.Parent = Main
    Corner(PreviewPanel, 10)
    local pst = Instance.new("UIStroke")
    pst.Color = STROKE
    pst.Thickness = 1
    pst.Transparency = 0.3
    pst.Parent = PreviewPanel

    local t = Txt(PreviewPanel, "ESP Preview", 13, TEXT)
    t.Font = Enum.Font.GothamBold
    t.Position = UDim2.new(0, 12, 0, 8)
    t.Size = UDim2.new(0, 100, 0, 20)

    local visBtn = Instance.new("TextButton")
    visBtn.Size = UDim2.new(0, 76, 0, 22)
    visBtn.Position = UDim2.new(1, -88, 0, 8)
    visBtn.BackgroundColor3 = OFF
    visBtn.BorderSizePixel = 0
    visBtn.Text = "Visivel"
    visBtn.Font = Enum.Font.GothamMedium
    visBtn.TextSize = 12
    visBtn.TextColor3 = TEXT
    visBtn.AutoButtonColor = false
    visBtn.Parent = PreviewPanel
    Corner(visBtn, 6)
    Ripple(visBtn)
    visBtn.MouseButton1Click:Connect(function()
        PVVisible = not PVVisible
        visBtn.Text = PVVisible and "Visivel" or "Invisivel"
        RefreshPreview()
    end)

    local Stage = Instance.new("Frame")
    Stage.BackgroundTransparency = 1
    Stage.Position = UDim2.new(0, 0, 0, 36)
    Stage.Size = UDim2.new(1, 0, 1, -36)
    Stage.ClipsDescendants = true
    Stage.Parent = PreviewPanel

    -- cenario: brilho suave atras do personagem
    local Glow = Instance.new("Frame")
    Glow.AnchorPoint = Vector2.new(0.5, 0.5)
    Glow.Position = UDim2.new(0.5, 0, 0.55, 0)
    Glow.Size = UDim2.new(0, 190, 0, 190)
    Glow.BackgroundColor3 = ACCENT
    Glow.BackgroundTransparency = 0.88
    Glow.BorderSizePixel = 0
    Glow.Parent = Stage
    Corner(Glow, 95)
    OnTheme(function(c) Tween(Glow, 0.4, { BackgroundColor3 = c }) end)
    PV.Glow = Glow

    local VF = Instance.new("ViewportFrame")
    VF.BackgroundTransparency = 1
    VF.Size = UDim2.new(1, 0, 1, 0)
    VF.Ambient = Color3.fromRGB(190, 190, 200)
    VF.LightColor = Color3.new(1, 1, 1)
    VF.LightDirection = Vector3.new(0.3, -0.5, 1)
    VF.Parent = Stage
    PV.VF = VF

    local World = Instance.new("WorldModel")
    World.Parent = VF
    PV.World = World

    local Cam = Instance.new("Camera")
    Cam.FieldOfView = 40
    Cam.Parent = VF
    VF.CurrentCamera = Cam
    PV.Cam = Cam

    local Loading = Txt(Stage, "Carregando sua skin...", 12, MUTED, Enum.TextXAlignment.Center)
    Loading.Size = UDim2.new(1, 0, 1, 0)
    PV.Loading = Loading

    local Overlay = Instance.new("Frame")
    Overlay.BackgroundTransparency = 1
    Overlay.Size = UDim2.new(1, 0, 1, 0)
    Overlay.ZIndex = 5
    Overlay.Parent = Stage

    local Box = Instance.new("Frame")
    Box.BackgroundTransparency = 1
    Box.ZIndex = 6
    Box.Visible = false
    Box.Parent = Overlay
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 1.5
    stroke.Parent = Box
    PV.Box, PV.BoxStroke = Box, stroke

    local HBg = Instance.new("Frame")
    HBg.BackgroundColor3 = Color3.new(0, 0, 0)
    HBg.BorderSizePixel = 0
    HBg.ZIndex = 6
    HBg.Visible = false
    HBg.Parent = Overlay
    local HFill = Instance.new("Frame")
    HFill.AnchorPoint = Vector2.new(0, 1)
    HFill.BorderSizePixel = 0
    HFill.Position = UDim2.new(0, 0, 1, 0)
    HFill.Size = UDim2.new(1, 0, 1, 0)
    HFill.ZIndex = 7
    HFill.Parent = HBg
    PV.HBg, PV.HFill = HBg, HFill

    local NameL = Txt(Overlay, LP.DisplayName, 13, TEXT, Enum.TextXAlignment.Center)
    NameL.Size = UDim2.new(0, 200, 0, 16)
    NameL.ZIndex = 7
    NameL.Font = Enum.Font.GothamBold
    NameL.Visible = false
    PV.Name = NameL

    local InfoL = Txt(Overlay, "", 12, TEXT, Enum.TextXAlignment.Center)
    InfoL.Size = UDim2.new(0, 200, 0, 16)
    InfoL.ZIndex = 7
    InfoL.Visible = false
    PV.Info = InfoL

    local function MakeLine(z)
        local f = Instance.new("Frame")
        f.AnchorPoint = Vector2.new(0.5, 0.5)
        f.BorderSizePixel = 0
        f.ZIndex = z
        f.Visible = false
        f.Parent = Overlay
        return f
    end
    PV.SetLine = function(f, a, b, thick, color)
        local d = b - a
        f.Size = UDim2.new(0, d.Magnitude, 0, thick)
        f.Position = UDim2.new(0, (a.X + b.X) / 2, 0, (a.Y + b.Y) / 2)
        f.Rotation = math.deg(math.atan2(d.Y, d.X))
        f.BackgroundColor3 = color
    end
    PV.Bones = {}
    for i = 1, 14 do PV.Bones[i] = MakeLine(7) end
    PV.Tracer = MakeLine(4)
end

-- projecao manual (3D do viewport -> pixels do palco)
local function Proj(pos)
    local cam = PV.Cam
    local rel = cam.CFrame:PointToObjectSpace(pos)
    local depth = math.max(-rel.Z, 0.01)
    local th = math.tan(math.rad(cam.FieldOfView / 2))
    local nx = rel.X / (depth * th * (STAGE_W / STAGE_H))
    local ny = rel.Y / (depth * th)
    return Vector2.new((0.5 + nx / 2) * STAGE_W, (0.5 - ny / 2) * STAGE_H)
end

local function SetAvatar(model)
    if PV.Model then PV.Model:Destroy() end
    PV.Model, PV.Head, PV.Root, PV.BonePairs = nil, nil, nil, nil

    for _, d in ipairs(model:GetDescendants()) do
        if d:IsA("Script") or d:IsA("LocalScript") or d:IsA("ModuleScript") or d:IsA("Sound")
            or d:IsA("Highlight") or d:IsA("BillboardGui") or d:IsA("ForceField") then
            d:Destroy()
        elseif d:IsA("BasePart") then
            d.Anchored = true
        end
    end
    local hum = model:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
        hum.HealthDisplayType = Enum.HumanoidHealthDisplayType.AlwaysOff
    end
    local root = model:FindFirstChild("HumanoidRootPart")
    if root and not model.PrimaryPart then model.PrimaryPart = root end

    model.Parent = PV.World
    model:PivotTo(CFrame.new(0, 0, 0))

    local cf, size = model:GetBoundingBox()
    local center = cf.Position
    local dist = size.Y / (0.62 * 2 * math.tan(math.rad(PV.Cam.FieldOfView / 2)))
    PV.Cam.CFrame = CFrame.lookAt(center + Vector3.new(0, 0, -dist), center)

    PV.Model = model
    PV.Head = model:FindFirstChild("Head")
    PV.Root = root
    local list = model:FindFirstChild("UpperTorso") and R15 or R6
    PV.BonePairs = {}
    for _, pr in ipairs(list) do
        local a, b = model:FindFirstChild(pr[1]), model:FindFirstChild(pr[2])
        if a and b then PV.BonePairs[#PV.BonePairs + 1] = { a, b } end
    end
    PV.Loading.Visible = false
end

local AvatarLoading = false
local function LoadAvatar()
    if AvatarLoading then return end
    AvatarLoading = true
    PV.Loading.Visible = true
    PV.Loading.Text = "Carregando sua skin..."
    task.spawn(function()
        local model
        local char = LP.Character
        if char then
            local ok, res = pcall(function()
                local was = char.Archivable
                char.Archivable = true
                local c = char:Clone()
                char.Archivable = was
                return c
            end)
            if ok then model = res end
        end
        if not model then
            local ok, res = pcall(function()
                local d = Players:GetHumanoidDescriptionFromUserId(LP.UserId)
                return Players:CreateHumanoidModelFromDescription(d, Enum.HumanoidRigType.R15)
            end)
            if ok then model = res end
        end
        if model then
            local ok = pcall(SetAvatar, model)
            if not ok then PV.Loading.Visible = true; PV.Loading.Text = "Skin indisponivel" end
        else
            PV.Loading.Text = "Skin indisponivel"
        end
        AvatarLoading = false
    end)
end

RefreshPreview = function()
    if not PV.Box then return end
    local on = S.ESP
    PV.Box.Visible = on and S.Box
    PV.BoxStroke.Color = PVVisible and S.ColorVisible or S.ColorHidden
    PV.HBg.Visible = on and S.Health
    PV.Name.Visible = on and S.Name
    PV.Name.TextColor3 = S.ColorESP

    local parts = {}
    if S.Distance then parts[#parts + 1] = "42m" end
    if S.Weapon then
        local tool = LP.Character and LP.Character:FindFirstChildOfClass("Tool")
        parts[#parts + 1] = tool and tool.Name or "Sem arma"
    end
    if S.Team then parts[#parts + 1] = LP.Team and LP.Team.Name or "Sem time" end
    PV.Info.Visible = on and #parts > 0
    PV.Info.Text = table.concat(parts, " | ")
    PV.Info.TextColor3 = S.ColorESP

    for i, ln in ipairs(PV.Bones) do
        ln.Visible = on and S.Skeleton and PV.BonePairs ~= nil and PV.BonePairs[i] ~= nil
        ln.BackgroundColor3 = S.ColorSkeleton
    end
    PV.Tracer.Visible = on and S.Tracer
    PV.Tracer.BackgroundColor3 = S.ColorTracer

    -- chams: tinge a skin do preview
    if on and S.Chams then
        Tween(PV.VF, 0.3, { Ambient = S.ColorChams, LightColor = S.ColorChams, ImageColor3 = S.ColorChams })
    else
        Tween(PV.VF, 0.3, { Ambient = Color3.fromRGB(190, 190, 200), LightColor = Color3.new(1, 1, 1), ImageColor3 = Color3.new(1, 1, 1) })
    end
end

local function UpdatePreviewAnim(t)
    if not (PreviewActive and MenuOpen and PV.Model) then return end
    local yaw = math.sin(t * 0.9) * 0.5
    PV.Model:PivotTo(CFrame.new(0, 0, 0) * CFrame.Angles(0, yaw, 0))
    PV.Glow.BackgroundTransparency = 0.88 + math.sin(t * 2) * 0.04

    if not S.ESP or not PV.Head or not PV.Root then return end
    local top = Proj(PV.Head.Position + Vector3.new(0, 0.6, 0))
    local bot = Proj(PV.Root.Position - Vector3.new(0, 3, 0))
    local c = Proj(PV.Root.Position)
    local h = math.abs(bot.Y - top.Y)
    local w = h * 0.55
    local x, y = c.X - w / 2, top.Y

    if S.Box then
        PV.Box.Position = UDim2.fromOffset(x, y)
        PV.Box.Size = UDim2.fromOffset(w, h)
    end
    if S.Health then
        local frac = 0.72 + 0.28 * math.sin(t * 1.2)
        PV.HBg.Position = UDim2.fromOffset(x - 9, y)
        PV.HBg.Size = UDim2.fromOffset(5, h)
        PV.HFill.Size = UDim2.new(1, 0, frac, 0)
        PV.HFill.BackgroundColor3 = Color3.fromRGB(255 * (1 - frac), 255 * frac, 0)
    end
    if S.Name then PV.Name.Position = UDim2.fromOffset(c.X - 100, y - 20) end
    if PV.Info.Visible then PV.Info.Position = UDim2.fromOffset(c.X - 100, y + h + 3) end
    if S.Skeleton then
        for i, pr in ipairs(PV.BonePairs) do
            local ln = PV.Bones[i]
            if ln then PV.SetLine(ln, Proj(pr[1].Position), Proj(pr[2].Position), 1.5, S.ColorSkeleton) end
        end
    end
    if S.Tracer then
        PV.SetLine(PV.Tracer, Vector2.new(STAGE_W / 2, STAGE_H), Vector2.new(c.X, bot.Y), 1.5, S.ColorTracer)
    end
end

--------------------------------------------------------------------
-- ABAS
--------------------------------------------------------------------
local PJogador = AddTab("Jogador")
local PVisuais = AddTab("Visuais")
local PAimbot = AddTab("Aimbot")
local PHitbox = AddTab("Hitbox")
local PExploits = AddTab("Exploits")
local PVeiculos = AddTab("Veiculos")
local PInv = AddTab("Inventario")
local PConfig = AddTab("Configuracoes")

-- caches (sem varrer o mapa toda hora)
local Prompts, Seats = {}, {}

local function ApplyPrompt(p)
    pcall(function()
        if S.RangeActive then p.MaxActivationDistance = S.PromptRange end
        if S.InstantPrompts then
            p.HoldDuration = 0
            p.Cooldown = 0
            p.RequiresLineOfSight = false
        end
    end)
end
local function ApplyAllPrompts()
    for p in pairs(Prompts) do
        if p.Parent then ApplyPrompt(p) else Prompts[p] = nil end
    end
end
local function Track(d)
    if d:IsA("ProximityPrompt") then
        Prompts[d] = true
        if S.InstantPrompts or S.RangeActive then ApplyPrompt(d) end
    elseif d:IsA("VehicleSeat") then
        Seats[d] = true
    end
end
Connect(Workspace.DescendantAdded, Track)
Connect(Workspace.DescendantRemoving, function(d)
    Prompts[d] = nil
    Seats[d] = nil
end)
task.spawn(function()
    local n = 0
    for _, d in ipairs(Workspace:GetDescendants()) do
        Track(d)
        n = n + 1
        if n % 500 == 0 then task.wait() end
    end
end)

-- JOGADOR -----------------------------------------------------------
local DefaultSpeed = 16
local SelectedPlayer = nil
Section(PJogador, "Movimento")
AddToggle(PJogador, "Velocidade", false, function(v)
    S.Speed = v
    local h = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if h then
        if v then DefaultSpeed = h.WalkSpeed else h.WalkSpeed = DefaultSpeed end
    end
end)
AddSlider(PJogador, "Valor da velocidade", 16, 200, 40, function(v) S.SpeedValue = v end)
AddToggle(PJogador, "Pulo infinito", false, function(v) S.InfJump = v end)
AddToggle(PJogador, "NoClip", false, function(v) S.NoClip = v end)

Section(PJogador, "Teleporte para jogadores")
local selBtn
selBtn = AddButton(PJogador, "Jogador: (clique para trocar)", function()
    local list = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP then list[#list + 1] = p end
    end
    if #list == 0 then
        selBtn.Text = "Jogador: ninguem no servidor"
        SelectedPlayer = nil
        return
    end
    local cur = table.find(list, SelectedPlayer) or 0
    SelectedPlayer = list[cur % #list + 1]
    selBtn.Text = "Jogador: " .. SelectedPlayer.Name
end)
AddButton(PJogador, "Teleportar para o jogador", function()
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    local t = SelectedPlayer and SelectedPlayer.Character and SelectedPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp and t then
        hrp.CFrame = t.CFrame * CFrame.new(0, 0, 3)
        Toast("Teleportado", "ok", "Para " .. SelectedPlayer.Name)
    else
        Toast("Selecione um jogador", "warn", "Escolha alguem na lista primeiro")
    end
end)
AddButton(PJogador, "Teleportar para o mais proximo", function()
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local best, bd, bp = nil, math.huge, nil
    for _, p in ipairs(Players:GetPlayers()) do
        local r = p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
        if r then
            local d = (r.Position - hrp.Position).Magnitude
            if d < bd then best, bd, bp = r, d, p end
        end
    end
    if best then
        hrp.CFrame = best.CFrame * CFrame.new(0, 0, 3)
        Toast("Teleportado", "ok", "Para " .. bp.Name)
    else
        Toast("Ninguem no servidor", "warn", "Nao ha jogadores para teleportar")
    end
end)

-- VISUAIS -----------------------------------------------------------
Section(PVisuais, "Funcoes")
AddToggle(PVisuais, "ESP Jogadores", false, function(v) S.ESP = v end)
AddToggle(PVisuais, "Box", false, function(v) S.Box = v end)
AddToggle(PVisuais, "Barra de Vida", false, function(v) S.Health = v end)
AddToggle(PVisuais, "Skeleton", false, function(v) S.Skeleton = v end)
AddToggle(PVisuais, "Nome", false, function(v) S.Name = v end)
AddToggle(PVisuais, "Distancia", false, function(v) S.Distance = v end)
AddToggle(PVisuais, "Arma", false, function(v) S.Weapon = v end)
AddToggle(PVisuais, "Time", false, function(v) S.Team = v end)
AddToggle(PVisuais, "Tracer", false, function(v) S.Tracer = v end)
AddToggle(PVisuais, "Ignorar Meu Time (Team Check)", false, function(v) S.TeamCheck = v end)
AddToggle(PVisuais, "Chams", false, function(v) S.Chams = v end)
AddSlider(PVisuais, "Distancia maxima", 100, 5000, 1500, function(v) S.MaxDist = v end, 50)
Section(PVisuais, "Cores (clique para trocar)")
AddColor(PVisuais, "Cor do ESP", "ColorESP")
AddColor(PVisuais, "Cor da Box (Visivel)", "ColorVisible")
AddColor(PVisuais, "Cor da Box (Invisivel)", "ColorHidden")
AddColor(PVisuais, "Cor do Tracer", "ColorTracer")
AddColor(PVisuais, "Cor do Skeleton", "ColorSkeleton")
AddColor(PVisuais, "Cor dos Chams", "ColorChams")
Section(PVisuais, "Preview")
AddButton(PVisuais, "Atualizar skin do preview", function()
    LoadAvatar()
    Toast("Skin", "info", "Atualizando o boneco do preview")
end)

-- AIMBOT ------------------------------------------------------------
Section(PAimbot, "Aimbot (segure o botao direito do mouse)")
AddToggle(PAimbot, "Aimbot", false, function(v) S.Aimbot = v end)
AddToggle(PAimbot, "Mostrar FOV", true, function(v) S.ShowFOV = v end)
AddToggle(PAimbot, "Checar parede", true, function(v) S.AimWall = v end)
AddToggle(PAimbot, "Ignorar meu time", false, function(v) S.AimTeam = v end)
AddSlider(PAimbot, "FOV", 30, 600, 150, function(v) S.AimFOV = v end, 5)
AddSlider(PAimbot, "Suavidade (menor = mais suave)", 0.05, 1, 0.25, function(v) S.AimSmooth = v end, 0.05)
AddCycle(PAimbot, "Alvo", { "Head", "UpperTorso", "HumanoidRootPart" }, "Head", function(v) S.AimPart = v end)

-- HITBOX ------------------------------------------------------------
local OrigSizes = {}
local function ResetHitboxes()
    for part, d in pairs(OrigSizes) do
        if part and part.Parent then
            pcall(function()
                part.Size = d.Size
                part.Transparency = d.Transparency
                part.CanCollide = d.CanCollide
            end)
        end
    end
    OrigSizes = {}
end

Section(PHitbox, "Expansor de hitbox")
AddToggle(PHitbox, "Hitbox", false, function(v)
    S.Hitbox = v
    if not v then ResetHitboxes() end
end)
AddSlider(PHitbox, "Tamanho", 2, 30, 8, function(v) S.HitboxSize = v end)
AddSlider(PHitbox, "Transparencia", 0, 1, 0.6, function(v) S.HitboxTransp = v end, 0.05)
AddToggle(PHitbox, "Afetar meu time", false, function(v) S.HitboxTeam = v end)

-- EXPLOITS ----------------------------------------------------------
Section(PExploits, "Prompts")
AddToggle(PExploits, "Instant Prompts (sem hold e sem cooldown)", false, function(v)
    S.InstantPrompts = v
    if v then ApplyAllPrompts() end
end)
AddSlider(PExploits, "Alcance dos prompts", 10, 1000, 20, function(v)
    S.PromptRange = v
    S.RangeActive = true
    ApplyAllPrompts()
end, 10)
AddToggle(PExploits, "Auto Interact (prompts no alcance)", false, function(v) S.AutoInteract = v end)
AddButton(PExploits, "Usar TODOS os prompts do mapa", function()
    local n = 0
    for p in pairs(Prompts) do
        if p.Parent and p.Enabled then
            n = n + 1
            pcall(fireproximityprompt, p, 0)
            task.wait(0.05)
        end
    end
    Toast("Prompts usados", "ok", n .. " prompts ativados")
end)

-- VEICULOS ----------------------------------------------------------
Section(PVeiculos, "Veiculos")
AddToggle(PVeiculos, "ESP Veiculos", false, function(v) S.ESPVehicles = v end)
AddButton(PVeiculos, "Teleportar para o veiculo mais proximo", function()
    local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local best, bd = nil, math.huge
    for seat in pairs(Seats) do
        if seat.Parent then
            local d = (seat.Position - hrp.Position).Magnitude
            if d < bd then best, bd = seat, d end
        end
    end
    if best then
        hrp.CFrame = best.CFrame * CFrame.new(0, 3, 0)
        Toast("Teleportado", "ok", "Voce esta no veiculo")
    else
        Toast("Sem veiculos", "warn", "Nenhum veiculo encontrado no mapa")
    end
end)

-- INVENTARIO --------------------------------------------------------
-- Dados reais: Tools da mochila (Backpack) e do personagem (equipados) +
-- leaderstats. Quantidade/categoria/descricao vem dos atributos ou valores do
-- proprio item (Quantity/Amount/Count..., Category/Type..., ToolTip/Description).
local invConns = {}
do
local InvHolder = PInv.Parent
PInv.Visible = false

local Inv = { Cells = {}, Groups = {}, Selected = nil, Cat = "Todos", Query = "", Sig = "", Cats = "" }

local InvRoot = Instance.new("Frame")
InvRoot.BackgroundTransparency = 1
InvRoot.Size = UDim2.new(1, 0, 1, 0)
InvRoot.Parent = InvHolder

local INV_LIST_W = 396

-- busca
local SearchWrap = Instance.new("Frame")
SearchWrap.Size = UDim2.new(0, INV_LIST_W, 0, 30)
SearchWrap.BackgroundColor3 = ITEM
SearchWrap.BorderSizePixel = 0
SearchWrap.Parent = InvRoot
Style(SearchWrap, false)
local SearchBox = Instance.new("TextBox")
SearchBox.BackgroundTransparency = 1
SearchBox.Position = UDim2.new(0, 12, 0, 0)
SearchBox.Size = UDim2.new(1, -24, 1, 0)
SearchBox.Text = ""
SearchBox.PlaceholderText = "Pesquisar item..."
SearchBox.PlaceholderColor3 = MUTED
SearchBox.TextColor3 = TEXT
SearchBox.Font = Enum.Font.Gotham
SearchBox.TextSize = 13
SearchBox.TextXAlignment = Enum.TextXAlignment.Left
SearchBox.ClearTextOnFocus = false
SearchBox.Parent = SearchWrap

-- categorias (chips)
local CatBar = Instance.new("ScrollingFrame")
CatBar.BackgroundTransparency = 1
CatBar.BorderSizePixel = 0
CatBar.Position = UDim2.new(0, 0, 0, 36)
CatBar.Size = UDim2.new(0, INV_LIST_W, 0, 28)
CatBar.CanvasSize = UDim2.new(0, 0, 0, 0)
CatBar.AutomaticCanvasSize = Enum.AutomaticSize.X
CatBar.ScrollBarThickness = 0
CatBar.ScrollingDirection = Enum.ScrollingDirection.X
CatBar.Parent = InvRoot
local CatLayout = Instance.new("UIListLayout")
CatLayout.FillDirection = Enum.FillDirection.Horizontal
CatLayout.Padding = UDim.new(0, 6)
CatLayout.VerticalAlignment = Enum.VerticalAlignment.Center
CatLayout.Parent = CatBar

-- grade de itens
local Grid = Instance.new("ScrollingFrame")
Grid.BackgroundTransparency = 1
Grid.BorderSizePixel = 0
Grid.Position = UDim2.new(0, 0, 0, 70)
Grid.Size = UDim2.new(0, INV_LIST_W + 8, 1, -70)
Grid.CanvasSize = UDim2.new(0, 0, 0, 0)
Grid.AutomaticCanvasSize = Enum.AutomaticSize.Y
Grid.ScrollBarThickness = 3
Grid.ScrollBarImageColor3 = ACCENT
Grid.Parent = InvRoot
OnTheme(function(c) Grid.ScrollBarImageColor3 = c end)
local GridLayout = Instance.new("UIGridLayout")
GridLayout.CellSize = UDim2.new(0, 92, 0, 98)
GridLayout.CellPadding = UDim2.new(0, 6, 0, 6)
GridLayout.SortOrder = Enum.SortOrder.Name
GridLayout.Parent = Grid

local EmptyLbl = Txt(InvRoot, "", 13, MUTED, Enum.TextXAlignment.Center)
EmptyLbl.Position = UDim2.new(0, 0, 0, 120)
EmptyLbl.Size = UDim2.new(0, INV_LIST_W, 0, 60)
EmptyLbl.TextWrapped = true
EmptyLbl.TextYAlignment = Enum.TextYAlignment.Top

local InvSpin, InvSpinTw = MakeSpinner(InvRoot, 18, 2)
InvSpin.Position = UDim2.new(0, INV_LIST_W - 22, 0, 0)
InvSpin.AnchorPoint = Vector2.new(0, 0)
InvSpin.Visible = false
InvSpin.Position = UDim2.new(0, INV_LIST_W + 2, 0, 6)

-- painel direito: resumo + detalhes
local Side2 = Instance.new("Frame")
Side2.BackgroundTransparency = 1
Side2.Position = UDim2.new(0, INV_LIST_W + 16, 0, 0)
Side2.Size = UDim2.new(1, -(INV_LIST_W + 16), 1, 0)
Side2.Parent = InvRoot

local Summary = Instance.new("Frame")
Summary.BackgroundColor3 = ITEM
Summary.BorderSizePixel = 0
Summary.Size = UDim2.new(1, 0, 0, 78)
Summary.Parent = Side2
Corner(Summary, 8)
local SumStroke = Instance.new("UIStroke")
SumStroke.Color = STROKE
SumStroke.Transparency = 0.35
SumStroke.Parent = Summary
local SumTitle = Txt(Summary, "RESUMO", 10, ACCENT)
SumTitle.Font = Enum.Font.GothamBold
SumTitle.Position = UDim2.new(0, 12, 0, 6)
SumTitle.Size = UDim2.new(1, -24, 0, 14)
OnTheme(function(c) SumTitle.TextColor3 = c end)
local SumLbl = Txt(Summary, "", 12, TEXT)
SumLbl.Position = UDim2.new(0, 12, 0, 22)
SumLbl.Size = UDim2.new(1, -24, 0, 52)
SumLbl.TextWrapped = true
SumLbl.TextYAlignment = Enum.TextYAlignment.Top

local Detail = Instance.new("CanvasGroup")
Detail.BackgroundColor3 = ITEM
Detail.BorderSizePixel = 0
Detail.Position = UDim2.new(0, 0, 0, 86)
Detail.Size = UDim2.new(1, 0, 1, -86)
Detail.Parent = Side2
Corner(Detail, 8)
local DetStroke = Instance.new("UIStroke")
DetStroke.Color = STROKE
DetStroke.Transparency = 0.35
DetStroke.Parent = Detail

local DetEmpty = Txt(Detail, "Clique em um item para ver os detalhes", 12, MUTED, Enum.TextXAlignment.Center)
DetEmpty.Size = UDim2.new(1, -20, 1, 0)
DetEmpty.Position = UDim2.new(0, 10, 0, 0)
DetEmpty.TextWrapped = true

local DetBody = Instance.new("Frame")
DetBody.BackgroundTransparency = 1
DetBody.Size = UDim2.new(1, 0, 1, 0)
DetBody.Visible = false
DetBody.Parent = Detail

local DetIconBG = Instance.new("Frame")
DetIconBG.BackgroundColor3 = BG
DetIconBG.BorderSizePixel = 0
DetIconBG.Position = UDim2.new(0, 12, 0, 12)
DetIconBG.Size = UDim2.new(0, 56, 0, 56)
DetIconBG.Parent = DetBody
Corner(DetIconBG, 10)
local DetIcon = Instance.new("ImageLabel")
DetIcon.BackgroundTransparency = 1
DetIcon.Position = UDim2.new(0, 4, 0, 4)
DetIcon.Size = UDim2.new(1, -8, 1, -8)
DetIcon.ScaleType = Enum.ScaleType.Fit
DetIcon.Parent = DetIconBG
local DetLetter = Txt(DetIconBG, "?", 22, ACCENT, Enum.TextXAlignment.Center)
DetLetter.Font = Enum.Font.GothamBlack
DetLetter.Size = UDim2.new(1, 0, 1, 0)
OnTheme(function(c) DetLetter.TextColor3 = c end)

local DetName = Txt(DetBody, "", 14, Color3.new(1, 1, 1))
DetName.Font = Enum.Font.GothamBold
DetName.Position = UDim2.new(0, 76, 0, 14)
DetName.Size = UDim2.new(1, -84, 0, 20)
DetName.TextTruncate = Enum.TextTruncate.AtEnd
local DetMeta = Txt(DetBody, "", 11, MUTED)
DetMeta.Position = UDim2.new(0, 76, 0, 34)
DetMeta.Size = UDim2.new(1, -84, 0, 14)
local DetState = Txt(DetBody, "", 11, ACCENT)
DetState.Font = Enum.Font.GothamBold
DetState.Position = UDim2.new(0, 76, 0, 50)
DetState.Size = UDim2.new(1, -84, 0, 14)
local DetDesc = Txt(DetBody, "", 12, TEXT)
DetDesc.Position = UDim2.new(0, 12, 0, 76)
DetDesc.Size = UDim2.new(1, -24, 0, 70)
DetDesc.TextWrapped = true
DetDesc.TextYAlignment = Enum.TextYAlignment.Top

local function InvAction(y, text)
    local b = Instance.new("TextButton")
    b.Position = UDim2.new(0, 12, 1, y)
    b.Size = UDim2.new(1, -24, 0, 28)
    b.BackgroundColor3 = ITEM
    b.BorderSizePixel = 0
    b.Text = text
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 13
    b.TextColor3 = TEXT
    b.AutoButtonColor = false
    b.Parent = DetBody
    Style(b, true)
    b.MouseEnter:Connect(function() Tween(b, 0.15, { TextSize = 14 }) end)
    b.MouseLeave:Connect(function() Tween(b, 0.15, { TextSize = 13 }) end)
    return b
end
local BtnEquip = InvAction(-130, "Equipar")
local BtnUse = InvAction(-96, "Usar")
local BtnCopy = InvAction(-62, "Copiar nome")

local function InvHumanoid()
    local c = LP.Character
    return c and c:FindFirstChildOfClass("Humanoid")
end

-- leitura dos dados reais de cada Tool
local QtyNames = { "Quantity", "Amount", "Count", "Stack", "Qty", "Quantidade" }
local CatNames = { "Category", "Categoria", "Type", "Tipo", "ItemType" }
local DescNames = { "Description", "Descricao", "Desc" }
local function ReadValue(tool, names, kind)
    for _, n in ipairs(names) do
        local a = tool:GetAttribute(n)
        if a ~= nil and type(a) == kind then return a end
        local v = tool:FindFirstChild(n)
        if v then
            if kind == "number" and (v:IsA("IntValue") or v:IsA("NumberValue")) then return v.Value end
            if kind == "string" and v:IsA("StringValue") and v.Value ~= "" then return v.Value end
        end
    end
    return nil
end

local function ScanInventory()
    local groups, order = {}, {}
    local function add(tool, equipped)
        local g = groups[tool.Name]
        if not g then
            g = { Name = tool.Name, Count = 0, Tools = {}, Equipped = false, Cat = "Itens", Icon = "", Desc = nil }
            groups[tool.Name] = g
            order[#order + 1] = tool.Name
        end
        g.Count = g.Count + (ReadValue(tool, QtyNames, "number") or 1)
        g.Tools[#g.Tools + 1] = tool
        if equipped then g.Equipped = true end
        if g.Icon == "" and tool.TextureId ~= "" then g.Icon = tool.TextureId end
        g.Cat = ReadValue(tool, CatNames, "string") or g.Cat
        g.Desc = g.Desc or (tool.ToolTip ~= "" and tool.ToolTip) or ReadValue(tool, DescNames, "string")
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, t in ipairs(bp:GetChildren()) do
            if t:IsA("Tool") then add(t, false) end
        end
    end
    local ch = LP.Character
    if ch then
        for _, t in ipairs(ch:GetChildren()) do
            if t:IsA("Tool") then add(t, true) end
        end
    end
    table.sort(order)
    return groups, order
end

local function CellMatches(g)
    if Inv.Cat == "Equipados" and not g.Equipped then return false end
    if Inv.Cat ~= "Todos" and Inv.Cat ~= "Equipados" and g.Cat ~= Inv.Cat then return false end
    if Inv.Query ~= "" and not string.find(string.lower(g.Name), Inv.Query, 1, true) then return false end
    return true
end

local function RefreshSummary()
    local total, eq, kinds = 0, 0, 0
    for _, g in pairs(Inv.Groups) do
        kinds = kinds + 1
        total = total + g.Count
        if g.Equipped then eq = eq + 1 end
    end
    local lines = { kinds .. " tipos | " .. total .. " itens | " .. eq .. " equipado(s)" }
    local ls = LP:FindFirstChild("leaderstats")
    if ls then
        local parts = {}
        for _, v in ipairs(ls:GetChildren()) do
            if v:IsA("ValueBase") then parts[#parts + 1] = v.Name .. ": " .. tostring(v.Value) end
        end
        if #parts > 0 then lines[#lines + 1] = table.concat(parts, "  |  ") end
    end
    SumLbl.Text = table.concat(lines, "\n")
end

local function RefreshDetail()
    local g = Inv.Selected and Inv.Groups[Inv.Selected]
    if not g then
        Inv.Selected = nil
        DetBody.Visible = false
        DetEmpty.Visible = true
        return
    end
    DetEmpty.Visible = false
    DetBody.Visible = true
    DetName.Text = g.Name
    DetMeta.Text = g.Cat .. "  |  Quantidade: " .. g.Count
    DetState.Text = g.Equipped and "EQUIPADO" or "NA MOCHILA"
    DetDesc.Text = g.Desc or "Este item nao tem descricao."
    if g.Icon ~= "" then
        DetIcon.Image = g.Icon
        DetIcon.Visible = true
        DetLetter.Visible = false
    else
        DetIcon.Visible = false
        DetLetter.Visible = true
        DetLetter.Text = string.upper(string.sub(g.Name, 1, 1))
    end
    BtnEquip.Text = g.Equipped and "Guardar na mochila" or "Equipar"
end

local function SelectItem(name)
    Inv.Selected = name
    for n, cell in pairs(Inv.Cells) do
        Tween(cell.Stroke, 0.2, { Color = n == name and ACCENT or STROKE, Transparency = n == name and 0 or 0.35 })
    end
    RefreshDetail()
    Detail.GroupTransparency = 0.6
    Tween(Detail, 0.3, { GroupTransparency = 0 })
    DetBody.Position = UDim2.new(0, 14, 0, 0)
    Tween(DetBody, 0.35, { Position = UDim2.new(0, 0, 0, 0) }, Enum.EasingStyle.Back)
end

local function MakeCell(g)
    local b = Instance.new("TextButton")
    b.Name = g.Name
    b.BackgroundColor3 = ITEM
    b.BorderSizePixel = 0
    b.Text = ""
    b.AutoButtonColor = false
    b.Parent = Grid
    Corner(b, 8)
    local st = Instance.new("UIStroke")
    st.Color = STROKE
    st.Transparency = 0.35
    st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    st.Parent = b
    local sc = Instance.new("UIScale")
    sc.Parent = b

    local img = Instance.new("ImageLabel")
    img.BackgroundTransparency = 1
    img.AnchorPoint = Vector2.new(0.5, 0)
    img.Position = UDim2.new(0.5, 0, 0, 8)
    img.Size = UDim2.new(0, 44, 0, 44)
    img.ScaleType = Enum.ScaleType.Fit
    img.Parent = b
    local letter = Txt(b, "", 22, ACCENT, Enum.TextXAlignment.Center)
    letter.Font = Enum.Font.GothamBlack
    letter.Position = UDim2.new(0, 0, 0, 8)
    letter.Size = UDim2.new(1, 0, 0, 44)
    local nm = Txt(b, g.Name, 11, TEXT, Enum.TextXAlignment.Center)
    nm.Position = UDim2.new(0, 4, 0, 56)
    nm.Size = UDim2.new(1, -8, 0, 32)
    nm.TextWrapped = true
    nm.TextTruncate = Enum.TextTruncate.AtEnd
    local badge = Txt(b, "", 10, OnAccent(ACCENT), Enum.TextXAlignment.Center)
    badge.Font = Enum.Font.GothamBold
    badge.BackgroundTransparency = 0
    badge.BackgroundColor3 = ACCENT
    badge.AnchorPoint = Vector2.new(1, 0)
    badge.Position = UDim2.new(1, -5, 0, 5)
    badge.Size = UDim2.new(0, 26, 0, 16)
    Corner(badge, 8)
    local dot = Instance.new("Frame")
    dot.BackgroundColor3 = Color3.fromRGB(70, 220, 130)
    dot.BorderSizePixel = 0
    dot.Position = UDim2.new(0, 7, 0, 7)
    dot.Size = UDim2.new(0, 8, 0, 8)
    dot.Parent = b
    Corner(dot, 4)

    OnTheme(function(c)
        letter.TextColor3 = c
        badge.BackgroundColor3 = c
        badge.TextColor3 = OnAccent(c)
    end)
    b.MouseEnter:Connect(function()
        Tween(b, 0.15, { BackgroundColor3 = HOVER })
        Tween(sc, 0.2, { Scale = 1.05 }, Enum.EasingStyle.Back)
    end)
    b.MouseLeave:Connect(function()
        Tween(b, 0.15, { BackgroundColor3 = ITEM })
        Tween(sc, 0.2, { Scale = 1 })
    end)
    b.MouseButton1Down:Connect(function() Tween(sc, 0.08, { Scale = 0.95 }) end)
    b.MouseButton1Click:Connect(function()
        Tween(sc, 0.2, { Scale = 1.05 }, Enum.EasingStyle.Back)
        SelectItem(g.Name)
    end)
    return { Btn = b, Img = img, Letter = letter, Badge = badge, Dot = dot, Stroke = st, Scale = sc }
end

local function PopCells()
    local i = 0
    for _, cell in pairs(Inv.Cells) do
        if cell.Btn.Visible then
            i = i + 1
            cell.Scale.Scale = 0.8
            cell.Btn.BackgroundTransparency = 1
            task.delay(math.min(i * 0.025, 0.5), function()
                if cell.Btn.Parent then
                    Tween(cell.Scale, 0.4, { Scale = 1 }, Enum.EasingStyle.Back)
                    Tween(cell.Btn, 0.3, { BackgroundTransparency = 0 })
                end
            end)
        end
    end
end

local function ApplyFilter()
    local shown = 0
    for name, cell in pairs(Inv.Cells) do
        local g = Inv.Groups[name]
        local vis = g and CellMatches(g) or false
        cell.Btn.Visible = vis
        if vis then shown = shown + 1 end
    end
    if shown == 0 then
        EmptyLbl.Text = next(Inv.Groups) and "Nenhum item encontrado nesse filtro."
            or "Sua mochila esta vazia.\nOs itens aparecem aqui assim que voce pegar algum."
    else
        EmptyLbl.Text = ""
    end
end

local CatChips = {}
local function RebuildCats()
    local set, list = {}, {}
    for _, g in pairs(Inv.Groups) do
        if not set[g.Cat] then set[g.Cat] = true; list[#list + 1] = g.Cat end
    end
    table.sort(list)
    table.insert(list, 1, "Equipados")
    table.insert(list, 1, "Todos")
    local sig = table.concat(list, "|")
    if sig == Inv.Cats then return end
    Inv.Cats = sig
    for _, c in pairs(CatChips) do c:Destroy() end
    CatChips = {}
    if not table.find(list, Inv.Cat) then Inv.Cat = "Todos" end
    for _, name in ipairs(list) do
        local c = Instance.new("TextButton")
        c.AutomaticSize = Enum.AutomaticSize.X
        c.Size = UDim2.new(0, 0, 0, 24)
        c.BackgroundColor3 = name == Inv.Cat and ACCENT or ITEM
        c.BorderSizePixel = 0
        c.Text = name
        c.Font = Enum.Font.GothamMedium
        c.TextSize = 12
        c.TextColor3 = name == Inv.Cat and OnAccent(ACCENT) or TEXT
        c.AutoButtonColor = false
        c.Parent = CatBar
        Corner(c, 12)
        local p = Instance.new("UIPadding")
        p.PaddingLeft = UDim.new(0, 12)
        p.PaddingRight = UDim.new(0, 12)
        p.Parent = c
        CatChips[name] = c
        c.MouseEnter:Connect(function()
            if Inv.Cat ~= name then Tween(c, 0.15, { BackgroundColor3 = HOVER }) end
        end)
        c.MouseLeave:Connect(function()
            if Inv.Cat ~= name then Tween(c, 0.15, { BackgroundColor3 = ITEM }) end
        end)
        c.MouseButton1Click:Connect(function()
            Inv.Cat = name
            for n, ch in pairs(CatChips) do
                local sel = n == name
                Tween(ch, 0.2, { BackgroundColor3 = sel and ACCENT or ITEM, TextColor3 = sel and OnAccent(ACCENT) or TEXT })
            end
            ApplyFilter()
            PopCells()
        end)
    end
end

local function RefreshInventory(force)
    local groups, order = ScanInventory()
    local parts = {}
    for _, n in ipairs(order) do
        local g = groups[n]
        parts[#parts + 1] = table.concat({ n, g.Count, tostring(g.Equipped), g.Cat, g.Icon, tostring(g.Desc) }, "~")
    end
    local sig = table.concat(parts, "|")
    if sig == Inv.Sig and not force then return false end
    Inv.Sig = sig
    Inv.Groups = groups

    for name, cell in pairs(Inv.Cells) do
        if not groups[name] then
            cell.Btn:Destroy()
            Inv.Cells[name] = nil
        end
    end
    for _, n in ipairs(order) do
        local g = groups[n]
        local cell = Inv.Cells[n]
        if not cell then
            cell = MakeCell(g)
            Inv.Cells[n] = cell
        end
        cell.Badge.Text = "x" .. g.Count
        cell.Badge.Visible = g.Count > 1
        cell.Dot.Visible = g.Equipped
        if g.Icon ~= "" then
            cell.Img.Image = g.Icon
            cell.Img.Visible = true
            cell.Letter.Visible = false
        else
            cell.Img.Visible = false
            cell.Letter.Visible = true
            cell.Letter.Text = string.upper(string.sub(n, 1, 1))
        end
    end
    RebuildCats()
    ApplyFilter()
    RefreshSummary()
    RefreshDetail()
    return true
end

OnInvShown = function()
    InvSpin.Visible = true
    RefreshInventory(true)
    PopCells()
    task.wait(0.5)
    InvSpin.Visible = false
end

SearchBox:GetPropertyChangedSignal("Text"):Connect(function()
    Inv.Query = string.lower(SearchBox.Text)
    ApplyFilter()
end)

-- acoes (usam as Tools reais do jogo)
local function PickTool(g)
    local bp = LP:FindFirstChildOfClass("Backpack")
    for _, t in ipairs(g.Tools) do
        if t.Parent and bp and t.Parent == bp then return t end
    end
    return g.Tools[1]
end

BtnEquip.MouseButton1Click:Connect(function()
    local g = Inv.Selected and Inv.Groups[Inv.Selected]
    local hum = InvHumanoid()
    if not g or not hum then return Toast("Personagem indisponivel", "err") end
    if g.Equipped then
        pcall(function() hum:UnequipTools() end)
        Toast(g.Name .. " guardado", "info")
    else
        local t = PickTool(g)
        if t and t.Parent then
            local ok = pcall(function() hum:EquipTool(t) end)
            Toast(ok and (g.Name .. " equipado") or "Nao foi possivel equipar", ok and "ok" or "err")
        end
    end
    task.delay(0.25, function() RefreshInventory() end)
end)

BtnUse.MouseButton1Click:Connect(function()
    local g = Inv.Selected and Inv.Groups[Inv.Selected]
    local hum = InvHumanoid()
    if not g or not hum then return Toast("Personagem indisponivel", "err") end
    local t = PickTool(g)
    if not (t and t.Parent) then return Toast("Item nao encontrado", "err") end
    task.spawn(function()
        if t.Parent ~= LP.Character then
            pcall(function() hum:EquipTool(t) end)
            task.wait(0.2)
        end
        local ok = pcall(function() t:Activate() end)
        Toast(ok and (g.Name .. " usado") or "Nao foi possivel usar", ok and "ok" or "err")
        task.wait(0.25)
        RefreshInventory()
    end)
end)

BtnCopy.MouseButton1Click:Connect(function()
    local g = Inv.Selected and Inv.Groups[Inv.Selected]
    if not g then return end
    local ok = pcall(function() setclipboard(g.Name) end)
    Toast(ok and "Nome copiado" or "Seu executor nao tem setclipboard", ok and "ok" or "warn")
end)

-- atualizacao automatica: eventos da mochila/personagem + checagem leve
local invPending = false
local function RequestInvRefresh()
    if invPending then return end
    invPending = true
    task.delay(0.15, function()
        invPending = false
        if Gui.Parent and Current == "Inventario" and MenuOpen then RefreshInventory() end
    end)
end
local function HookInventory()
    for _, c in ipairs(invConns) do pcall(function() c:Disconnect() end) end
    invConns = {}
    local bp = LP:FindFirstChildOfClass("Backpack")
    for _, inst in ipairs({ bp, LP.Character }) do
        if inst then
            invConns[#invConns + 1] = inst.ChildAdded:Connect(RequestInvRefresh)
            invConns[#invConns + 1] = inst.ChildRemoved:Connect(RequestInvRefresh)
        end
    end
end
HookInventory()
Connect(LP.CharacterAdded, function()
    task.wait(0.5)
    HookInventory()
    Inv.Sig = ""
    RequestInvRefresh()
end)
Connect(LP.ChildAdded, function(c)
    if c:IsA("Backpack") then HookInventory(); Inv.Sig = ""; RequestInvRefresh() end
end)
task.spawn(function()
    while Gui.Parent do
        task.wait(1.5)
        if Current == "Inventario" and MenuOpen then pcall(RefreshInventory) end
    end
end)

end -- fim do bloco do inventario

-- CONFIG ------------------------------------------------------------
local Unload
local ThemeNames = {}
for _, t in ipairs(Themes) do ThemeNames[#ThemeNames + 1] = t[1] end
Section(PConfig, "Aparencia")
AddCycle(PConfig, "Cor do tema", ThemeNames, CurrentTheme, function(name)
    for _, t in ipairs(Themes) do
        if t[1] == name then
            ApplyTheme(t[2])
            pcall(writefile, "BHHub_theme.txt", name)
        end
    end
end)

-- imagem fixa do menu (controlada SO pelo dono) ------------------------
-- Nao existe campo para o usuario trocar a imagem. Ela vem do arquivo remoto BRAND_URL,
-- que so o dono edita. O ultimo valor valido fica em cache (BHHub_brand.txt) para
-- funcionar offline e permanecer ate o dono trocar o arquivo remoto.
local function IsOwner()
    for _, id in ipairs(OWNER_IDS) do
        if id == LP.UserId then return true end
    end
    return false
end

-- banner fixo no canto superior esquerdo (embutido no script: igual para todos,
-- sem opcao de troca no menu). Gravado como arquivo e carregado com getcustomasset.
task.spawn(function()
    local LOGO = [==[
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
]==]
    local chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    local map = {}
    for i = 1, 64 do map[string.byte(chars, i)] = i - 1 end
    local function decode(str)
        local out, n, buf, bits = {}, 0, 0, 0
        for i = 1, #str do
            local v = map[string.byte(str, i)]
            if v then
                buf = buf * 64 + v
                bits = bits + 6
                if bits >= 8 then
                    bits = bits - 8
                    local d = 2 ^ bits
                    local byte = math.floor(buf / d)
                    buf = buf - byte * d
                    n = n + 1
                    out[n] = string.char(byte)
                end
            end
        end
        return table.concat(out)
    end
    local ok, err = pcall(function()
        local raw = decode(LOGO)
        writefile("BHHub_logo.jpg", raw)
        local asset = getcustomasset("BHHub_logo.jpg")
        IconImg.Image = asset
        IconImg.Visible = true
        IconFallback.Visible = false
    end)
    if not ok then warn("BH Hub: banner nao carregado (" .. tostring(err) .. ")") end
end)

local BrandApplied = nil
local function ApplyBrand(src)
    src = tostring(src or ""):gsub("%s+", "")
    if src == "" or src == BrandApplied then return false end
    local asset
    if src:match("^%d+$") then
        asset = "rbxassetid://" .. src
    elseif src:match("^rbxassetid://") then
        asset = src
    elseif src:match("^https://") and writefile and getcustomasset then
        local ok, data = pcall(function() return game:HttpGet(src) end)
        if ok and data and #data > 0 then
            local ok2, a = pcall(function()
                writefile("BHHub_brand.png", data)
                return getcustomasset("BHHub_brand.png")
            end)
            if ok2 then asset = a end
        end
    end
    if not asset then return false end
    BrandApplied = src
    IconImg.Image = asset
    IconImg.Visible = true
    IconFallback.Visible = false
    return true
end

local function FetchBrand()
    if BRAND_URL == "" or not BRAND_URL:match("^https://") then return end
    local ok, txt = pcall(function() return game:HttpGet(BRAND_URL) end)
    if ok and txt and #txt > 0 and #txt < 600 then
        local line = txt:match("[^\r\n]+")
        if line and ApplyBrand(line) then
            pcall(writefile, "BHHub_brand.txt", BrandApplied)
        end
    end
end

pcall(function()
    if isfile and isfile("BHHub_brand.txt") then ApplyBrand(readfile("BHHub_brand.txt")) end
end)
task.spawn(function()
    while Gui.Parent do
        FetchBrand()
        task.wait(180)
    end
end)

Section(PConfig, "Imagem do menu")
AddButton(PConfig, "Imagem fixa definida pelo dono", function() end)
if IsOwner() then
    Section(PConfig, "Dono")
    AddButton(PConfig, "Atualizar imagem agora", function()
        if not IsOwner() then return Toast("Sem permissao", "err", "Somente o dono pode usar isto") end
        if BRAND_URL == "" then return Toast("BRAND_URL vazio", "warn", "Defina o link no topo do script") end
        task.spawn(function()
            BrandApplied = nil
            FetchBrand()
            Toast("Imagem atualizada", "ok", "Carregada do arquivo do dono", "brand")
        end)
    end)
end
AddToggle(PConfig, "Sons das notificacoes", true, function(v) Sfx.On = v end)
AddToggle(PConfig, "Desfocar o jogo com o menu aberto", false, function(v)
    BlurOn = v
    if MenuOpen then Tween(Blur, 0.3, { Size = v and 12 or 0 }) end
end)
AddButton(PConfig, "Testar notificacoes", function()
    Toast("Tudo certo por aqui", "ok")
    task.delay(0.15, function() Toast("Isto e um aviso", "warn") end)
    task.delay(0.3, function() Toast("Isto e um erro", "err") end)
end)
Section(PConfig, "Tecla do menu")
local keyBtn
local function RefreshKeyBtn()
    if keyBtn then
        keyBtn.Text = "Tecla para abrir/fechar: " .. MenuKey.Name .. " (clique para mudar)"
    end
    HintLbl.Text = "Menu: " .. MenuKey.Name
end
keyBtn = AddButton(PConfig, "", function()
    if ListeningKey then return end
    ListeningKey = true
    keyBtn.Text = "Pressione uma tecla... (Esc cancela)"
end)
RefreshKeyBtn()
AddButton(PConfig, "Restaurar tecla padrao (LeftAlt)", function()
    MenuKey = Enum.KeyCode.LeftAlt
    ListeningKey = false
    pcall(writefile, "BHHub_menukey.txt", MenuKey.Name)
    RefreshKeyBtn()
    Toast("Tecla do menu", "ok", "Restaurada para LeftAlt")
end)
Section(PConfig, "Geral")
AddButton(PConfig, "Fechar menu", function() SetMenu(false) end)
AddButton(PConfig, "Descarregar script", function() Unload() end)

--------------------------------------------------------------------
-- ESP NO JOGO (otimizado)
--------------------------------------------------------------------
local ESP, Highlights = {}, {}

-- escreve na propriedade do Drawing SO se o valor mudou (cada escrita e uma chamada nativa cara)
local function P(o, sh, k, v)
    if sh[k] ~= v then
        sh[k] = v
        o[k] = v
    end
end

local function CreateESP(p)
    local e = {
        Box = NewDraw("Square", { Thickness = 1, Filled = false }),
        Tracer = NewDraw("Line", { Thickness = 1 }),
        Name = NewDraw("Text", { Size = 13, Center = true, Outline = true }),
        Info = NewDraw("Text", { Size = 12, Center = true, Outline = true }),
        HBg = NewDraw("Line", { Thickness = 4, Color = Color3.new(0, 0, 0) }),
        HBar = NewDraw("Line", { Thickness = 2 }),
        Skel = {},
        Hidden = true,
        -- copias dos ultimos valores escritos (veja P)
        sBox = {}, sTracer = {}, sName = {}, sInfo = {}, sHBg = {}, sHBar = {}, sSkel = {},
    }
    for i = 1, 14 do
        e.Skel[i] = NewDraw("Line", { Thickness = 1 })
        e.sSkel[i] = {}
    end
    ESP[p] = e
end

local function HideESP(e)
    if e.Hidden then return end
    e.Hidden = true
    P(e.Box, e.sBox, "Visible", false)
    P(e.Tracer, e.sTracer, "Visible", false)
    P(e.Name, e.sName, "Visible", false)
    P(e.Info, e.sInfo, "Visible", false)
    P(e.HBg, e.sHBg, "Visible", false)
    P(e.HBar, e.sHBar, "Visible", false)
    for i, l in ipairs(e.Skel) do P(l, e.sSkel[i], "Visible", false) end
end

local function RemoveESP(p)
    local e = ESP[p]
    if e then
        for _, k in ipairs({ "Box", "Tracer", "Name", "Info", "HBg", "HBar" }) do
            pcall(function() e[k]:Remove() end)
        end
        for _, l in ipairs(e.Skel) do pcall(function() l:Remove() end) end
        ESP[p] = nil
    end
    if Highlights[p] then
        pcall(function() Highlights[p]:Destroy() end)
        Highlights[p] = nil
    end
end

for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LP then CreateESP(p) end
end
Connect(Players.PlayerAdded, function(p) if p ~= LP then CreateESP(p) end end)
Connect(Players.PlayerRemoving, RemoveESP)

local function SameTeam(p)
    return p.Team ~= nil and LP.Team ~= nil and p.Team == LP.Team
end

local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
local VisCache = setmetatable({}, { __mode = "k" })
local function IsVisible(part, char)
    local c = VisCache[part]
    local now = os.clock()
    if c and now - c.t < 0.12 then return c.v end
    rayParams.FilterDescendantsInstances = { LP.Character, Camera }
    local o = Camera.CFrame.Position
    local r = Workspace:Raycast(o, part.Position - o, rayParams)
    local v = (r == nil) or r.Instance:IsDescendantOf(char)
    VisCache[part] = { t = now, v = v }
    return v
end

local function SetChams(p, char, on)
    local h = Highlights[p]
    if on and char then
        if not h or h.Parent ~= char then
            if h then h:Destroy() end
            h = Instance.new("Highlight")
            h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            h.FillTransparency = 0.5
            h.OutlineTransparency = 0
            h.Parent = char
            Highlights[p] = h
        end
        if h.FillColor ~= S.ColorChams then
            h.FillColor = S.ColorChams
            h.OutlineColor = S.ColorChams
        end
    elseif h then
        h:Destroy()
        Highlights[p] = nil
    end
end

local ESPAllHidden = false
local function UpdateESP()
    if not S.ESP then
        if not ESPAllHidden then
            ESPAllHidden = true
            for p, e in pairs(ESP) do
                HideESP(e)
                SetChams(p, nil, false)
            end
        end
        return
    end
    ESPAllHidden = false

    local myRoot = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    local camPos = Camera.CFrame.Position
    local vp = Camera.ViewportSize

    for p, e in pairs(ESP) do
        local char = p.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local root = char and char:FindFirstChild("HumanoidRootPart")
        local head = char and char:FindFirstChild("Head")

        if not (hum and root and head) or hum.Health <= 0 or (S.TeamCheck and SameTeam(p)) then
            HideESP(e)
            SetChams(p, nil, false)
        else
            local dist = (root.Position - (myRoot and myRoot.Position or camPos)).Magnitude
            if dist > S.MaxDist then
                HideESP(e)
                SetChams(p, nil, false)
            else
                local c3 = Camera:WorldToViewportPoint(root.Position)
                if c3.Z <= 0 then
                    HideESP(e)
                else
                    e.Hidden = false
                    SetChams(p, char, S.Chams)

                    local top = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.6, 0))
                    local bot = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))
                    local h = math.abs(bot.Y - top.Y)
                    local w = h * 0.55
                    local x, y = c3.X - w / 2, top.Y

                    if S.Box then
                        P(e.Box, e.sBox, "Visible", true)
                        P(e.Box, e.sBox, "Size", Vector2.new(w, h))
                        P(e.Box, e.sBox, "Position", Vector2.new(x, y))
                        P(e.Box, e.sBox, "Color", IsVisible(head, char) and S.ColorVisible or S.ColorHidden)
                    else
                        P(e.Box, e.sBox, "Visible", false)
                    end

                    if S.Tracer then
                        P(e.Tracer, e.sTracer, "Visible", true)
                        P(e.Tracer, e.sTracer, "From", Vector2.new(vp.X / 2, vp.Y))
                        P(e.Tracer, e.sTracer, "To", Vector2.new(c3.X, bot.Y))
                        P(e.Tracer, e.sTracer, "Color", S.ColorTracer)
                    else
                        P(e.Tracer, e.sTracer, "Visible", false)
                    end

                    if S.Name then
                        P(e.Name, e.sName, "Visible", true)
                        P(e.Name, e.sName, "Text", p.DisplayName)
                        P(e.Name, e.sName, "Position", Vector2.new(c3.X, y - 16))
                        P(e.Name, e.sName, "Color", S.ColorESP)
                    else
                        P(e.Name, e.sName, "Visible", false)
                    end

                    if S.Distance or S.Weapon or S.Team then
                        local txt = ""
                        if S.Distance then txt = math.floor(dist) .. "m" end
                        if S.Weapon then
                            local tool = char:FindFirstChildOfClass("Tool")
                            txt = txt .. (txt ~= "" and " | " or "") .. (tool and tool.Name or "Sem arma")
                        end
                        if S.Team then
                            txt = txt .. (txt ~= "" and " | " or "") .. (p.Team and p.Team.Name or "Sem time")
                        end
                        P(e.Info, e.sInfo, "Visible", true)
                        P(e.Info, e.sInfo, "Text", txt)
                        P(e.Info, e.sInfo, "Position", Vector2.new(c3.X, y + h + 2))
                        P(e.Info, e.sInfo, "Color", S.ColorESP)
                    else
                        P(e.Info, e.sInfo, "Visible", false)
                    end

                    if S.Health then
                        local frac = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
                        frac = math.floor(frac * 50) / 50 -- evita reescrever a cor a cada decimo de vida
                        P(e.HBg, e.sHBg, "Visible", true)
                        P(e.HBg, e.sHBg, "From", Vector2.new(x - 5, y + h))
                        P(e.HBg, e.sHBg, "To", Vector2.new(x - 5, y))
                        P(e.HBar, e.sHBar, "Visible", true)
                        P(e.HBar, e.sHBar, "From", Vector2.new(x - 5, y + h))
                        P(e.HBar, e.sHBar, "To", Vector2.new(x - 5, y + h - h * frac))
                        P(e.HBar, e.sHBar, "Color", Color3.fromRGB(255 * (1 - frac), 255 * frac, 0))
                    else
                        P(e.HBg, e.sHBg, "Visible", false)
                        P(e.HBar, e.sHBar, "Visible", false)
                    end

                    if S.Skeleton then
                        local list = char:FindFirstChild("UpperTorso") and R15 or R6
                        for i, l in ipairs(e.Skel) do
                            local sh = e.sSkel[i]
                            local pr = list[i]
                            local a = pr and char:FindFirstChild(pr[1])
                            local b = pr and char:FindFirstChild(pr[2])
                            if a and b then
                                local pa = Camera:WorldToViewportPoint(a.Position)
                                local pb = Camera:WorldToViewportPoint(b.Position)
                                if pa.Z > 0 and pb.Z > 0 then
                                    P(l, sh, "Visible", true)
                                    P(l, sh, "From", Vector2.new(pa.X, pa.Y))
                                    P(l, sh, "To", Vector2.new(pb.X, pb.Y))
                                    P(l, sh, "Color", S.ColorSkeleton)
                                else
                                    P(l, sh, "Visible", false)
                                end
                            else
                                P(l, sh, "Visible", false)
                            end
                        end
                    else
                        for i, l in ipairs(e.Skel) do P(l, e.sSkel[i], "Visible", false) end
                    end
                end
            end
        end
    end
end

local VehDraws = {}
local VehHidden = true
local function UpdateVehicles()
    if not S.ESPVehicles then
        if not VehHidden then
            VehHidden = true
            for _, d in pairs(VehDraws) do d.Visible = false end
        end
        return
    end
    VehHidden = false
    local myRoot = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    for seat in pairs(Seats) do
        if seat.Parent then
            local t = VehDraws[seat]
            if not t then
                t = NewDraw("Text", { Size = 13, Center = true, Outline = true, Color = Color3.fromRGB(255, 220, 0) })
                VehDraws[seat] = t
            end
            local pos = Camera:WorldToViewportPoint(seat.Position)
            local dist = myRoot and (seat.Position - myRoot.Position).Magnitude or 0
            if pos.Z > 0 and dist <= S.MaxDist then
                local m = seat:FindFirstAncestorOfClass("Model")
                local label = (m and m.Name or "Veiculo") .. " [" .. math.floor(dist) .. "m]"
                if t.Text ~= label then t.Text = label end
                t.Position = Vector2.new(pos.X, pos.Y)
                t.Visible = true
            else
                t.Visible = false
            end
        end
    end
    for seat, t in pairs(VehDraws) do
        if not seat.Parent then
            pcall(function() t:Remove() end)
            VehDraws[seat] = nil
        end
    end
end

--------------------------------------------------------------------
-- AIMBOT
--------------------------------------------------------------------
local FOVCircle = NewDraw("Circle", { Thickness = 1, NumSides = 64, Filled = false, Color = Color3.new(1, 1, 1) })
local fovShown = false

local function GetTarget()
    local center = UserInputService:GetMouseLocation()
    local best, bd = nil, S.AimFOV
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LP and p.Character and not (S.AimTeam and SameTeam(p)) then
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            local part = p.Character:FindFirstChild(S.AimPart)
            if hum and hum.Health > 0 and part then
                local v = Camera:WorldToViewportPoint(part.Position)
                if v.Z > 0 then
                    local d = (Vector2.new(v.X, v.Y) - center).Magnitude
                    if d < bd and (not S.AimWall or IsVisible(part, p.Character)) then
                        best, bd = part, d
                    end
                end
            end
        end
    end
    return best
end

--------------------------------------------------------------------
-- LOOPS
--------------------------------------------------------------------
local fpsFrames, fpsTime = 0, 0
local lastScaleCheck, lastBg = 0, 0
Connect(RunService.RenderStepped, function(dt)
    Camera = Workspace.CurrentCamera
    local now = os.clock()

    UpdateESP()
    UpdateVehicles()
    UpdatePreviewAnim(now)

    -- animacoes de fundo do menu (so com o menu aberto)
    if Shadow.Visible then
        Shadow.Position = Main.Position
        ShadowScale.Scale = MainScale.Scale
    end
    if MenuOpen then
        -- responsividade: reajusta a escala se a janela mudar de tamanho
        if now - lastScaleCheck > 0.5 then
            lastScaleCheck = now
            local target = BaseScale()
            if math.abs(MainScale.Scale - target) > 0.01 and Main.GroupTransparency < 0.05 then
                Tween(MainScale, 0.25, { Scale = target })
            end
        end
        if now - lastBg > 0.05 then -- 20 atualizacoes/s bastam e poupam CPU
            lastBg = now
            LogoGrad.Offset = Vector2.new(math.sin(now * 1.5) * 0.6, 0)
            MainGrad.Rotation = 45 + math.sin(now * 0.4) * 25
            MainStroke.Transparency = 0.5 + math.sin(now * 2) * 0.2
        end

        fpsFrames = fpsFrames + 1
        fpsTime = fpsTime + dt
        if fpsTime >= 0.5 then
            local ping = 0
            pcall(function() ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) end)
            StatsLbl.Text = math.floor(fpsFrames / fpsTime) .. " FPS | " .. ping .. "ms"
            fpsFrames, fpsTime = 0, 0
        end
    end

    local showFov = S.Aimbot and S.ShowFOV
    if showFov then
        FOVCircle.Radius = S.AimFOV
        FOVCircle.Position = UserInputService:GetMouseLocation()
        FOVCircle.Visible = true
        fovShown = true
    elseif fovShown then
        FOVCircle.Visible = false
        fovShown = false
    end

    if S.Aimbot and not MenuOpen and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local t = GetTarget()
        if t then
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, t.Position), S.AimSmooth)
        end
    end
end)

local accHit, accAuto = 0, 0
Connect(RunService.Heartbeat, function(dt)
    local char = LP.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum and S.Speed and hum.WalkSpeed ~= S.SpeedValue then
        hum.WalkSpeed = S.SpeedValue
    end

    accHit = accHit + dt
    if accHit >= 0.25 then
        accHit = 0
        if S.Hitbox then
            local sz = Vector3.new(S.HitboxSize, S.HitboxSize, S.HitboxSize)
            for _, p in ipairs(Players:GetPlayers()) do
                local root = p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
                if root then
                    if SameTeam(p) and not S.HitboxTeam then
                        local o = OrigSizes[root]
                        if o then
                            root.Size = o.Size
                            root.Transparency = o.Transparency
                            OrigSizes[root] = nil
                        end
                    else
                        if not OrigSizes[root] then
                            OrigSizes[root] = { Size = root.Size, Transparency = root.Transparency, CanCollide = root.CanCollide }
                        end
                        if root.Size ~= sz then root.Size = sz end
                        root.Transparency = S.HitboxTransp
                        root.CanCollide = false
                    end
                end
            end
        end
    end

    accAuto = accAuto + dt
    if accAuto >= 0.25 then
        accAuto = 0
        if S.AutoInteract then
            local root = char and char:FindFirstChild("HumanoidRootPart")
            if root then
                for p in pairs(Prompts) do
                    if p.Parent and p.Enabled then
                        local par = p.Parent
                        local part = par:IsA("BasePart") and par or par:FindFirstChildWhichIsA("BasePart")
                        if part and (part.Position - root.Position).Magnitude <= S.PromptRange then
                            pcall(fireproximityprompt, p, 0)
                            break
                        end
                    end
                end
            end
        end
    end
end)

Connect(RunService.Stepped, function()
    if S.NoClip and LP.Character then
        for _, d in ipairs(LP.Character:GetChildren()) do
            if d:IsA("BasePart") and d.CanCollide then d.CanCollide = false end
        end
    end
end)

Connect(UserInputService.JumpRequest, function()
    if S.InfJump then
        local h = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

Connect(UserInputService.InputBegan, function(i, gp)
    if i.UserInputType ~= Enum.UserInputType.Keyboard then return end

    -- modo "escutando": a proxima tecla vira a tecla do menu
    if ListeningKey then
        ListeningKey = false
        if i.KeyCode == Enum.KeyCode.Escape then
            RefreshKeyBtn()
            Toast("Cancelado", "off", "Tecla do menu mantida")
        elseif i.KeyCode ~= Enum.KeyCode.Unknown then
            MenuKey = i.KeyCode
            pcall(writefile, "BHHub_menukey.txt", MenuKey.Name)
            RefreshKeyBtn()
            Toast("Tecla do menu", "ok", "Agora: " .. MenuKey.Name)
        end
        return
    end

    if not gp and i.KeyCode == MenuKey and Gui.Parent and not KeyScreenOpen then
        SetMenu(not MenuOpen)
    end
end)

-- recarrega a skin do preview ao renascer
Connect(LP.CharacterAdded, function()
    task.wait(2)
    if Gui.Parent then LoadAvatar() end
end)

Unload = function()
    Toast("Script descarregado", "off", "Ate a proxima!")
    task.wait(0.4)
    for _, c in ipairs(Connections) do pcall(function() c:Disconnect() end) end
    for _, c in ipairs(invConns) do pcall(function() c:Disconnect() end) end
    pcall(function() Blur:Destroy() end)
    pcall(function() Shadow:Destroy() end)
    pcall(function()
        local h = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if h and S.Speed then h.WalkSpeed = DefaultSpeed end
    end)
    for p in pairs(ESP) do RemoveESP(p) end
    for _, h in pairs(Highlights) do pcall(function() h:Destroy() end) end
    for _, d in pairs(VehDraws) do pcall(function() d:Remove() end) end
    pcall(function() FOVCircle:Remove() end)
    ResetHitboxes()
    Gui:Destroy()
end

--------------------------------------------------------------------
-- TELA DE KEY (animada)
--------------------------------------------------------------------
local KeyFrame = Instance.new("CanvasGroup")
KeyFrame.AnchorPoint = Vector2.new(0.5, 0.5)
KeyFrame.Size = UDim2.new(0, 330, 0, 190)
KeyFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
KeyFrame.BackgroundColor3 = BG
KeyFrame.BorderSizePixel = 0
KeyFrame.GroupTransparency = 1
KeyFrame.Parent = Gui
Corner(KeyFrame, 10)
local KeyScale = Instance.new("UIScale")
KeyScale.Scale = 0.8
KeyScale.Parent = KeyFrame
local KeyBG = Instance.new("Frame")
KeyBG.Size = UDim2.new(1, 0, 1, 0)
KeyBG.BackgroundColor3 = Color3.new(1, 1, 1)
KeyBG.BorderSizePixel = 0
KeyBG.ZIndex = 0
KeyBG.Parent = KeyFrame
local KGrad = Instance.new("UIGradient")
KGrad.Color = ColorSequence.new(BG, BG2)
KGrad.Rotation = 45
KGrad.Parent = KeyBG
local KStroke = Instance.new("UIStroke")
KStroke.Color = ACCENT
KStroke.Thickness = 1.5
KStroke.Transparency = 0.4
KStroke.Parent = KeyFrame
Draggable(KeyFrame, KeyFrame)

local KT = Txt(KeyFrame, "BH HUB", 22, Color3.new(1, 1, 1), Enum.TextXAlignment.Center)
KT.Font = Enum.Font.GothamBlack
KT.Size = UDim2.new(1, 0, 0, 44)
local KTGrad = Instance.new("UIGradient")
KTGrad.Color = LogoGrad.Color
KTGrad.Parent = KT
local KSub = Txt(KeyFrame, "Digite a key para continuar", 12, MUTED, Enum.TextXAlignment.Center)
KSub.Position = UDim2.new(0, 0, 0, 40)
KSub.Size = UDim2.new(1, 0, 0, 16)

local KeyBox = Instance.new("TextBox")
KeyBox.Size = UDim2.new(1, -40, 0, 36)
KeyBox.Position = UDim2.new(0, 20, 0, 68)
KeyBox.BackgroundColor3 = ITEM
KeyBox.BorderSizePixel = 0
KeyBox.PlaceholderText = "Key"
KeyBox.PlaceholderColor3 = MUTED
KeyBox.Text = ""
KeyBox.TextColor3 = TEXT
KeyBox.Font = Enum.Font.Gotham
KeyBox.TextSize = 14
KeyBox.ClearTextOnFocus = false
KeyBox.Parent = KeyFrame
Corner(KeyBox, 7)
local KBoxStroke = Instance.new("UIStroke")
KBoxStroke.Color = STROKE
KBoxStroke.Thickness = 1
KBoxStroke.Parent = KeyBox
KeyBox.Focused:Connect(function() Tween(KBoxStroke, 0.2, { Color = ACCENT }) end)
KeyBox.FocusLost:Connect(function() Tween(KBoxStroke, 0.2, { Color = STROKE }) end)

local KeyMsg = Txt(KeyFrame, "", 12, Color3.fromRGB(255, 90, 90), Enum.TextXAlignment.Center)
KeyMsg.Position = UDim2.new(0, 0, 0, 108)
KeyMsg.Size = UDim2.new(1, 0, 0, 16)

local KeyBtn = Instance.new("TextButton")
KeyBtn.Size = UDim2.new(1, -40, 0, 36)
KeyBtn.Position = UDim2.new(0, 20, 0, 134)
KeyBtn.BackgroundColor3 = ACCENT
KeyBtn.BorderSizePixel = 0
KeyBtn.Text = "Verificar"
KeyBtn.Font = Enum.Font.GothamBold
KeyBtn.TextSize = 14
KeyBtn.TextColor3 = OnAccent(ACCENT)
KeyBtn.AutoButtonColor = false
KeyBtn.Parent = KeyFrame
Corner(KeyBtn, 7)
KeyBtn.MouseEnter:Connect(function() Tween(KeyBtn, 0.15, { BackgroundColor3 = ACCENT:Lerp(Color3.new(1, 1, 1), 0.25) }) end)
KeyBtn.MouseLeave:Connect(function() Tween(KeyBtn, 0.15, { BackgroundColor3 = ACCENT }) end)

Tween(KeyFrame, 0.45, { GroupTransparency = 0 })
Tween(KeyScale, 0.55, { Scale = 1 }, Enum.EasingStyle.Back)

local keyAnim = Connect(RunService.RenderStepped, function()
    local now = os.clock()
    KTGrad.Offset = Vector2.new(math.sin(now * 1.5) * 0.6, 0)
    KGrad.Rotation = 45 + math.sin(now * 0.5) * 25
    KStroke.Transparency = 0.4 + math.sin(now * 2) * 0.2
end)

local checking = false
local function CheckKey()
    if checking then return end
    if KeyBox.Text == KEY then
        checking = true
        KeyMsg.TextColor3 = Color3.fromRGB(90, 255, 140)
        KeyMsg.Text = "Key valida!"
        Tween(KeyFrame, 0.4, { GroupTransparency = 1 })
        Tween(KeyScale, 0.4, { Scale = 0.9 })
        task.delay(0.45, function()
            keyAnim:Disconnect()
            KeyFrame:Destroy()
            KeyScreenOpen = false
            ShowPage("Visuais", true)
            SetMenu(true)
            LoadAvatar()
            Toast("Bem-vindo, " .. LP.DisplayName .. "!", "ok")
        end)
    else
        KeyMsg.Text = "Key incorreta"
        KeyBox.Text = ""
        local base = KeyFrame.Position
        for _, dx in ipairs({ 10, -10, 6, -6, 0 }) do
            Tween(KeyFrame, 0.05, { Position = UDim2.new(base.X.Scale, base.X.Offset + dx, base.Y.Scale, base.Y.Offset) }, Enum.EasingStyle.Linear)
            task.wait(0.05)
        end
    end
end
KeyBtn.MouseButton1Click:Connect(CheckKey)
KeyBox.FocusLost:Connect(function(enter) if enter then CheckKey() end end)

RefreshPreview()
print("BH Hub v5 carregado | Key: ??? | tecla do menu: " .. MenuKey.Name .. " (configuravel em Configuracoes)")
