-- BH Hub v3 | Belo Horizonte MG | Xeno
-- Key: ??? | RightShift abre/fecha o menu
-- v3: preview com a SUA skin, animacoes em todo o menu, temas e toasts

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
local KEY = "FLAD"

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
local MenuKey = Enum.KeyCode.RightShift
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
    for k, v in pairs(props or {}) do d[k] = v end
    return d
end

--------------------------------------------------------------------
-- TEMA / CORES
--------------------------------------------------------------------
local BG = Color3.fromRGB(14, 17, 25)
local BG2 = Color3.fromRGB(19, 23, 36)
local SIDE = Color3.fromRGB(11, 14, 21)
local ITEM = Color3.fromRGB(22, 26, 38)
local HOVER = Color3.fromRGB(32, 38, 56)
local OFF = Color3.fromRGB(45, 50, 66)
local STROKE = Color3.fromRGB(42, 48, 68)
local TEXT = Color3.fromRGB(215, 220, 230)
local MUTED = Color3.fromRGB(130, 138, 155)
local ACCENT = Color3.fromRGB(0, 120, 215)

local ThemeHooks = {}
local function OnTheme(fn) ThemeHooks[#ThemeHooks + 1] = fn end
local function ApplyTheme(c)
    ACCENT = c
    for _, fn in ipairs(ThemeHooks) do pcall(fn, c) end
end
local Themes = {
    { "Azul", Color3.fromRGB(0, 120, 215) }, { "Roxo", Color3.fromRGB(140, 90, 255) },
    { "Verde", Color3.fromRGB(0, 200, 120) }, { "Vermelho", Color3.fromRGB(235, 60, 70) },
    { "Laranja", Color3.fromRGB(255, 140, 30) }, { "Rosa", Color3.fromRGB(255, 90, 170) },
}

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

-- toasts (avisos animados)
local function Toast(text)
    if not Gui.Parent then return end
    local t = Instance.new("CanvasGroup")
    t.AnchorPoint = Vector2.new(0.5, 1)
    t.Size = UDim2.new(0, 260, 0, 34)
    t.Position = UDim2.new(0.5, 0, 1, 10)
    t.BackgroundColor3 = BG2
    t.GroupTransparency = 1
    t.BorderSizePixel = 0
    t.Parent = Gui
    Corner(t, 8)
    local st = Instance.new("UIStroke")
    st.Color = ACCENT
    st.Thickness = 1
    st.Transparency = 0.3
    st.Parent = t
    local l = Txt(t, text, 13, TEXT, Enum.TextXAlignment.Center)
    l.Size = UDim2.new(1, 0, 1, 0)
    Tween(t, 0.35, { Position = UDim2.new(0.5, 0, 1, -70), GroupTransparency = 0 }, Enum.EasingStyle.Back)
    task.delay(2.2, function()
        Tween(t, 0.3, { Position = UDim2.new(0.5, 0, 1, -40), GroupTransparency = 1 })
        task.delay(0.35, function() t:Destroy() end)
    end)
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

local Logo = Txt(Side, "BH HUB", 24, Color3.new(1, 1, 1), Enum.TextXAlignment.Center)
Logo.Font = Enum.Font.GothamBlack
Logo.Size = UDim2.new(1, 0, 0, 50)
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
TabHolder.Position = UDim2.new(0, 8, 0, 56)
TabHolder.Size = UDim2.new(1, -16, 1, -116)
TabHolder.Parent = Side
local tabLayout = Instance.new("UIListLayout")
tabLayout.Padding = UDim.new(0, 4)
tabLayout.Parent = TabHolder

local Indicator = Instance.new("Frame")
Indicator.Size = UDim2.new(0, 3, 0, 20)
Indicator.Position = UDim2.new(0, 0, 0, 62)
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
local function SetMenu(v)
    MenuOpen = v
    if v then
        Main.Visible = true
        Tween(Main, 0.35, { GroupTransparency = 0 })
        Tween(MainScale, 0.45, { Scale = 1 }, Enum.EasingStyle.Back)
    else
        Tween(Main, 0.2, { GroupTransparency = 1 })
        Tween(MainScale, 0.2, { Scale = 0.92 })
        task.delay(0.22, function()
            if not MenuOpen then Main.Visible = false end
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
    end
    Tween(Indicator, 0.35, { Position = UDim2.new(0, 0, 0, 56 + (TabIndex[name] - 1) * 36 + 6) }, Enum.EasingStyle.Back)

    TitleLbl.Text = name
    TitleLbl.TextTransparency = 1
    TitleLbl.Position = UDim2.new(0, 28, 0, 0)
    Tween(TitleLbl, 0.35, { TextTransparency = 0, Position = UDim2.new(0, 16, 0, 0) })
end
OnTheme(function()
    if Current and TabButtons[Current] then TabButtons[Current].TextColor3 = ACCENT end
end)

local function AddTab(name)
    TabCount = TabCount + 1
    TabIndex[name] = TabCount

    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, 0, 0, 32)
    b.BackgroundColor3 = SIDE
    b.BorderSizePixel = 0
    b.Text = "   " .. name
    b.TextXAlignment = Enum.TextXAlignment.Left
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 13
    b.TextColor3 = MUTED
    b.AutoButtonColor = false
    b.Parent = TabHolder
    Corner(b, 6)

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
    dot.BackgroundColor3 = Color3.new(1, 1, 1)
    dot.BorderSizePixel = 0
    dot.Parent = box
    Corner(dot, 3)

    local state = default
    OnTheme(function() Tween(box, 0.25, { BackgroundColor3 = state and ACCENT or OFF }) end)
    f.MouseButton1Click:Connect(function()
        state = not state
        Tween(box, 0.2, { BackgroundColor3 = state and ACCENT or OFF })
        Tween(dot, 0.25, { Size = state and UDim2.new(0, 8, 0, 8) or UDim2.new(0, 0, 0, 0) }, Enum.EasingStyle.Back)
        Tween(l, 0.2, { TextColor3 = state and Color3.new(1, 1, 1) or TEXT })
        cb(state)
        Changed()
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
    b.MouseButton1Click:Connect(function() cb(b) end)
    return b
end

local function AddCycle(page, text, options, default, cb)
    local idx = table.find(options, default) or 1
    local b = AddButton(page, text .. ": " .. tostring(options[idx]), function() end)
    b.MouseButton1Click:Connect(function()
        idx = idx % #options + 1
        b.Text = text .. ": " .. tostring(options[idx])
        cb(options[idx])
        Changed()
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
        Toast("Teleportado para " .. SelectedPlayer.Name)
    else
        Toast("Selecione um jogador primeiro")
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
        Toast("Teleportado para " .. bp.Name)
    else
        Toast("Ninguem no servidor")
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
    Toast("Atualizando skin...")
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
    Toast("Prompts usados: " .. n)
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
        Toast("Teleportado para o veiculo")
    else
        Toast("Nenhum veiculo encontrado")
    end
end)

-- CONFIG ------------------------------------------------------------
local Unload
local ThemeNames = {}
for _, t in ipairs(Themes) do ThemeNames[#ThemeNames + 1] = t[1] end
Section(PConfig, "Aparencia")
AddCycle(PConfig, "Cor do tema", ThemeNames, "Azul", function(name)
    for _, t in ipairs(Themes) do
        if t[1] == name then ApplyTheme(t[2]) end
    end
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
AddButton(PConfig, "Restaurar tecla padrao (RightShift)", function()
    MenuKey = Enum.KeyCode.RightShift
    ListeningKey = false
    RefreshKeyBtn()
    Toast("Tecla do menu: RightShift")
end)
Section(PConfig, "Geral")
AddButton(PConfig, "Fechar menu", function() SetMenu(false) end)
AddButton(PConfig, "Descarregar script", function() Unload() end)

--------------------------------------------------------------------
-- ESP NO JOGO (otimizado)
--------------------------------------------------------------------
local ESP, Highlights = {}, {}

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
    }
    for i = 1, 14 do e.Skel[i] = NewDraw("Line", { Thickness = 1 }) end
    ESP[p] = e
end

local function HideESP(e)
    if e.Hidden then return end
    e.Hidden = true
    e.Box.Visible = false; e.Tracer.Visible = false; e.Name.Visible = false
    e.Info.Visible = false; e.HBg.Visible = false; e.HBar.Visible = false
    for _, l in ipairs(e.Skel) do l.Visible = false end
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
        h.FillColor = S.ColorChams
        h.OutlineColor = S.ColorChams
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
                        e.Box.Visible = true
                        e.Box.Size = Vector2.new(w, h)
                        e.Box.Position = Vector2.new(x, y)
                        e.Box.Color = IsVisible(head, char) and S.ColorVisible or S.ColorHidden
                    else
                        e.Box.Visible = false
                    end

                    if S.Tracer then
                        e.Tracer.Visible = true
                        e.Tracer.From = Vector2.new(vp.X / 2, vp.Y)
                        e.Tracer.To = Vector2.new(c3.X, bot.Y)
                        e.Tracer.Color = S.ColorTracer
                    else
                        e.Tracer.Visible = false
                    end

                    if S.Name then
                        e.Name.Visible = true
                        e.Name.Text = p.DisplayName
                        e.Name.Position = Vector2.new(c3.X, y - 16)
                        e.Name.Color = S.ColorESP
                    else
                        e.Name.Visible = false
                    end

                    if S.Distance or S.Weapon or S.Team then
                        local parts = {}
                        if S.Distance then parts[#parts + 1] = math.floor(dist) .. "m" end
                        if S.Weapon then
                            local tool = char:FindFirstChildOfClass("Tool")
                            parts[#parts + 1] = tool and tool.Name or "Sem arma"
                        end
                        if S.Team then parts[#parts + 1] = p.Team and p.Team.Name or "Sem time" end
                        e.Info.Visible = true
                        e.Info.Text = table.concat(parts, " | ")
                        e.Info.Position = Vector2.new(c3.X, y + h + 2)
                        e.Info.Color = S.ColorESP
                    else
                        e.Info.Visible = false
                    end

                    if S.Health then
                        local frac = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
                        e.HBg.Visible = true
                        e.HBg.From = Vector2.new(x - 5, y + h)
                        e.HBg.To = Vector2.new(x - 5, y)
                        e.HBar.Visible = true
                        e.HBar.From = Vector2.new(x - 5, y + h)
                        e.HBar.To = Vector2.new(x - 5, y + h - h * frac)
                        e.HBar.Color = Color3.fromRGB(255 * (1 - frac), 255 * frac, 0)
                    else
                        e.HBg.Visible = false
                        e.HBar.Visible = false
                    end

                    if S.Skeleton then
                        local list = char:FindFirstChild("UpperTorso") and R15 or R6
                        for i, l in ipairs(e.Skel) do
                            local pr = list[i]
                            local a = pr and char:FindFirstChild(pr[1])
                            local b = pr and char:FindFirstChild(pr[2])
                            if a and b then
                                local pa = Camera:WorldToViewportPoint(a.Position)
                                local pb = Camera:WorldToViewportPoint(b.Position)
                                if pa.Z > 0 and pb.Z > 0 then
                                    l.Visible = true
                                    l.From = Vector2.new(pa.X, pa.Y)
                                    l.To = Vector2.new(pb.X, pb.Y)
                                    l.Color = S.ColorSkeleton
                                else
                                    l.Visible = false
                                end
                            else
                                l.Visible = false
                            end
                        end
                    else
                        for _, l in ipairs(e.Skel) do l.Visible = false end
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
                t.Visible = true
                t.Position = Vector2.new(pos.X, pos.Y)
                t.Text = (m and m.Name or "Veiculo") .. " [" .. math.floor(dist) .. "m]"
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
    local center = Camera.ViewportSize / 2
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
Connect(RunService.RenderStepped, function(dt)
    Camera = Workspace.CurrentCamera
    local now = os.clock()

    UpdateESP()
    UpdateVehicles()
    UpdatePreviewAnim(now)

    -- animacoes de fundo do menu (so com o menu aberto)
    if MenuOpen then
        LogoGrad.Offset = Vector2.new(math.sin(now * 1.5) * 0.6, 0)
        MainGrad.Rotation = 45 + math.sin(now * 0.4) * 25
        MainStroke.Transparency = 0.5 + math.sin(now * 2) * 0.2

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
        FOVCircle.Visible = true
        FOVCircle.Radius = S.AimFOV
        FOVCircle.Position = Camera.ViewportSize / 2
        fovShown = true
    elseif fovShown then
        FOVCircle.Visible = false
        fovShown = false
    end

    if S.Aimbot and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
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
            Toast("Cancelado")
        elseif i.KeyCode ~= Enum.KeyCode.Unknown then
            MenuKey = i.KeyCode
            RefreshKeyBtn()
            Toast("Tecla do menu: " .. MenuKey.Name)
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
    Toast("Script descarregado")
    task.wait(0.4)
    for _, c in ipairs(Connections) do pcall(function() c:Disconnect() end) end
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
KeyBtn.TextColor3 = Color3.new(1, 1, 1)
KeyBtn.AutoButtonColor = false
KeyBtn.Parent = KeyFrame
Corner(KeyBtn, 7)
KeyBtn.MouseEnter:Connect(function() Tween(KeyBtn, 0.15, { BackgroundColor3 = Color3.fromRGB(30, 145, 240) }) end)
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
            Toast("Bem-vindo, " .. LP.DisplayName .. "!")
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
print("BH Hub v3 carregado | Key: HFDEGH-FJHKWA-32GHJD | tecla do menu: " .. MenuKey.Name .. " (configuravel em Configuracoes)")
