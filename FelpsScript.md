-- MENU CUSTOMIZADO DO PERERELPS VIP
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera

-- KEY SYSTEM
local CORRECT_KEY = "Lobozkssk2"
local keyEntered = false

-- CRIAR UI DE KEY
local KeyScreenGui = Instance.new("ScreenGui")
KeyScreenGui.Name = "PererelpsKeySystem"
KeyScreenGui.ResetOnSpawn = false
KeyScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

if gethui then
    KeyScreenGui.Parent = gethui()
elseif syn and syn.protect_gui then
    syn.protect_gui(KeyScreenGui)
    KeyScreenGui.Parent = game.CoreGui
else
    KeyScreenGui.Parent = game.CoreGui
end

local KeyFrame = Instance.new("Frame")
KeyFrame.Name = "KeyFrame"
KeyFrame.Size = UDim2.new(0, 400, 0, 250)
KeyFrame.Position = UDim2.new(0.5, -200, 0.5, -125)
KeyFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
KeyFrame.BorderSizePixel = 0
KeyFrame.Parent = KeyScreenGui

local KeyCorner = Instance.new("UICorner")
KeyCorner.CornerRadius = UDim.new(0, 12)
KeyCorner.Parent = KeyFrame

local KeyShadow = Instance.new("ImageLabel")
KeyShadow.Size = UDim2.new(1, 30, 1, 30)
KeyShadow.Position = UDim2.new(0, -15, 0, -15)
KeyShadow.BackgroundTransparency = 1
KeyShadow.Image = "rbxassetid://1316045217"
KeyShadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
KeyShadow.ImageTransparency = 0.6
KeyShadow.ScaleType = Enum.ScaleType.Slice
KeyShadow.SliceCenter = Rect.new(10, 10, 118, 118)
KeyShadow.ZIndex = 0
KeyShadow.Parent = KeyFrame

local KeyTopBar = Instance.new("Frame")
KeyTopBar.Size = UDim2.new(1, 0, 0, 50)
KeyTopBar.BackgroundColor3 = Color3.fromRGB(255, 0, 150)
KeyTopBar.BorderSizePixel = 0
KeyTopBar.Parent = KeyFrame

local KeyTopCorner = Instance.new("UICorner")
KeyTopCorner.CornerRadius = UDim.new(0, 12)
KeyTopCorner.Parent = KeyTopBar

local KeyTopBottom = Instance.new("Frame")
KeyTopBottom.Size = UDim2.new(1, 0, 0, 12)
KeyTopBottom.Position = UDim2.new(0, 0, 1, -12)
KeyTopBottom.BackgroundColor3 = Color3.fromRGB(255, 0, 150)
KeyTopBottom.BorderSizePixel = 0
KeyTopBottom.Parent = KeyTopBar

local KeyTitle = Instance.new("TextLabel")
KeyTitle.Size = UDim2.new(1, 0, 1, 0)
KeyTitle.BackgroundTransparency = 1
KeyTitle.Text = "🛑 ACESSO RESTRITO 🛑"
KeyTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
KeyTitle.TextSize = 20
KeyTitle.Font = Enum.Font.GothamBold
KeyTitle.Parent = KeyTopBar

local KeySubtitle = Instance.new("TextLabel")
KeySubtitle.Size = UDim2.new(1, -40, 0, 30)
KeySubtitle.Position = UDim2.new(0, 20, 0, 60)
KeySubtitle.BackgroundTransparency = 1
KeySubtitle.Text = "💎 MENU DO PERERELPS VIP 💎"
KeySubtitle.TextColor3 = Color3.fromRGB(200, 200, 200)
KeySubtitle.TextSize = 16
KeySubtitle.Font = Enum.Font.GothamBold
KeySubtitle.Parent = KeyFrame

local KeyNote = Instance.new("TextLabel")
KeyNote.Size = UDim2.new(1, -40, 0, 25)
KeyNote.Position = UDim2.new(0, 20, 0, 95)
KeyNote.BackgroundTransparency = 1
KeyNote.Text = "Digite a chave de acesso abaixo:"
KeyNote.TextColor3 = Color3.fromRGB(150, 150, 150)
KeyNote.TextSize = 13
KeyNote.Font = Enum.Font.Gotham
KeyNote.Parent = KeyFrame

local KeyInput = Instance.new("TextBox")
KeyInput.Size = UDim2.new(1, -40, 0, 40)
KeyInput.Position = UDim2.new(0, 20, 0, 130)
KeyInput.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
KeyInput.PlaceholderText = "Digite a chave aqui..."
KeyInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
KeyInput.Text = ""
KeyInput.TextColor3 = Color3.fromRGB(255, 255, 255)
KeyInput.TextSize = 14
KeyInput.Font = Enum.Font.Gotham
KeyInput.ClearTextOnFocus = false
KeyInput.Parent = KeyFrame

local KeyInputCorner = Instance.new("UICorner")
KeyInputCorner.CornerRadius = UDim.new(0, 8)
KeyInputCorner.Parent = KeyInput

local SubmitButton = Instance.new("TextButton")
SubmitButton.Size = UDim2.new(1, -40, 0, 40)
SubmitButton.Position = UDim2.new(0, 20, 0, 185)
SubmitButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
SubmitButton.Text = "✓ VALIDAR CHAVE"
SubmitButton.TextColor3 = Color3.fromRGB(255, 255, 255)
SubmitButton.TextSize = 15
SubmitButton.Font = Enum.Font.GothamBold
SubmitButton.Parent = KeyFrame

local SubmitCorner = Instance.new("UICorner")
SubmitCorner.CornerRadius = UDim.new(0, 8)
SubmitCorner.Parent = SubmitButton

local ErrorLabel = Instance.new("TextLabel")
ErrorLabel.Size = UDim2.new(1, -40, 0, 20)
ErrorLabel.Position = UDim2.new(0, 20, 0, 175)
ErrorLabel.BackgroundTransparency = 1
ErrorLabel.Text = ""
ErrorLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
ErrorLabel.TextSize = 12
ErrorLabel.Font = Enum.Font.Gotham
ErrorLabel.Visible = false
ErrorLabel.Parent = KeyFrame

-- Função para validar key
local function validateKey()
    local enteredKey = KeyInput.Text
    
    if enteredKey == CORRECT_KEY then
        keyEntered = true
        
        -- Animação de sucesso
        TweenService:Create(SubmitButton, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(50, 255, 50)
        }):Play()
        
        SubmitButton.Text = "✓ CHAVE CORRETA!"
        
        wait(0.5)
        
        -- Fade out
        TweenService:Create(KeyFrame, TweenInfo.new(0.5), {
            BackgroundTransparency = 1
        }):Play()
        
        for _, child in pairs(KeyFrame:GetDescendants()) do
            if child:IsA("TextLabel") or child:IsA("TextButton") or child:IsA("TextBox") then
                TweenService:Create(child, TweenInfo.new(0.5), {
                    TextTransparency = 1
                }):Play()
            elseif child:IsA("Frame") then
                TweenService:Create(child, TweenInfo.new(0.5), {
                    BackgroundTransparency = 1
                }):Play()
            elseif child:IsA("ImageLabel") then
                TweenService:Create(child, TweenInfo.new(0.5), {
                    ImageTransparency = 1
                }):Play()
            end
        end
        
        wait(0.5)
        KeyScreenGui:Destroy()
        
        -- Carregar menu principal
        loadMainMenu()
    else
        -- Animação de erro
        ErrorLabel.Text = "❌ Chave incorreta!"
        ErrorLabel.Visible = true
        
        TweenService:Create(SubmitButton, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        }):Play()
        
        TweenService:Create(KeyInput, TweenInfo.new(0.1), {
            BackgroundColor3 = Color3.fromRGB(80, 20, 20)
        }):Play()
        
        wait(0.3)
        
        TweenService:Create(SubmitButton, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        }):Play()
        
        TweenService:Create(KeyInput, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(30, 30, 35)
        }):Play()
        
        wait(1)
        ErrorLabel.Visible = false
    end
end

SubmitButton.MouseButton1Click:Connect(validateKey)

KeyInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        validateKey()
    end
end)

-- Função para carregar menu principal
function loadMainMenu()
    
-- Configurações
local _G = {
    AimbotEnabled = false,
    TeamCheck = false,
    FovRadius = 150,
    FovVisible = true,
    TargetPart = "Head",
    AimbotSmooth = 0.5,
    ESPEnabled = false,
    ESPBoxes = false,
    ESPNames = false,
    ESPDistance = false,
    ESPHealthBar = false,
    ESPTracers = false,
    FOVColor = Color3.fromRGB(255, 0, 150),
    ESPBoxColor = Color3.fromRGB(255, 255, 255),
    ESPNameColor = Color3.fromRGB(255, 255, 255),
    TracerColor = Color3.fromRGB(255, 255, 255)
}

-- ESP System
local function CreateESP(player)
    local drawings = {
        Box = Drawing.new("Square"),
        BoxOutline = Drawing.new("Square"),
        Name = Drawing.new("Text"),
        Distance = Drawing.new("Text"),
        HealthBar = Drawing.new("Square"),
        HealthBarOutline = Drawing.new("Square"),
        Tracer = Drawing.new("Line")
    }
    
    drawings.Box.Thickness = 2
    drawings.Box.Filled = false
    drawings.Box.Visible = false
    drawings.Box.ZIndex = 2
    
    drawings.BoxOutline.Thickness = 3
    drawings.BoxOutline.Filled = false
    drawings.BoxOutline.Color = Color3.fromRGB(0, 0, 0)
    drawings.BoxOutline.Visible = false
    drawings.BoxOutline.ZIndex = 1
    
    drawings.Name.Size = 14
    drawings.Name.Center = true
    drawings.Name.Outline = true
    drawings.Name.Visible = false
    drawings.Name.ZIndex = 2
    
    drawings.Distance.Size = 13
    drawings.Distance.Center = true
    drawings.Distance.Outline = true
    drawings.Distance.Visible = false
    drawings.Distance.ZIndex = 2
    
    drawings.HealthBar.Filled = true
    drawings.HealthBar.Thickness = 1
    drawings.HealthBar.Visible = false
    drawings.HealthBar.ZIndex = 3
    
    drawings.HealthBarOutline.Filled = false
    drawings.HealthBarOutline.Thickness = 1
    drawings.HealthBarOutline.Color = Color3.fromRGB(0, 0, 0)
    drawings.HealthBarOutline.Visible = false
    drawings.HealthBarOutline.ZIndex = 2
    
    drawings.Tracer.Thickness = 1.5
    drawings.Tracer.Visible = false
    drawings.Tracer.ZIndex = 2
    
    return drawings
end

local PlayerVisuals = {}
for _, p in pairs(Players:GetPlayers()) do 
    if p ~= LocalPlayer then 
        PlayerVisuals[p] = CreateESP(p) 
    end 
end

Players.PlayerAdded:Connect(function(p) 
    PlayerVisuals[p] = CreateESP(p) 
end)

Players.PlayerRemoving:Connect(function(p) 
    if PlayerVisuals[p] then 
        for _, drawing in pairs(PlayerVisuals[p]) do
            drawing:Remove()
        end
        PlayerVisuals[p] = nil 
    end 
end)

local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Transparency = 1
FOVCircle.NumSides = 32

-- LOOP DE ATUALIZAÇÃO
RunService:BindToRenderStep("PererelpsUpdate", 200, function()
    FOVCircle.Radius = _G.FovRadius
    FOVCircle.Visible = (_G.AimbotEnabled and _G.FovVisible)
    FOVCircle.Position = UserInputService:GetMouseLocation()
    FOVCircle.Color = _G.FOVColor

    if _G.AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = nil
        local dist = _G.FovRadius
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild(_G.TargetPart) then
                if _G.TeamCheck and p.Team == LocalPlayer.Team then continue end
                local pos, vis = Camera:WorldToViewportPoint(p.Character[_G.TargetPart].Position)
                if vis then
                    local mag = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                    if mag < dist then 
                        target = p.Character[_G.TargetPart] 
                        dist = mag 
                    end
                end
            end
        end
        if target then 
            local targetCFrame = CFrame.lookAt(Camera.CFrame.Position, target.Position)
            Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, _G.AimbotSmooth)
        end
    end

    for player, visual in pairs(PlayerVisuals) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and 
           player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            
            local root = player.Character.HumanoidRootPart
            local head = player.Character:FindFirstChild("Head")
            local humanoid = player.Character.Humanoid
            
            if not head then continue end
            
            local rootPos, rootVis = Camera:WorldToViewportPoint(root.Position)
            local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
            local legPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))
            
            local isEnemy = (not _G.TeamCheck or player.Team ~= LocalPlayer.Team)
            
            if rootVis and isEnemy and _G.ESPEnabled then
                if _G.ESPBoxes then
                    local height = math.abs(headPos.Y - legPos.Y)
                    local width = height / 2
                    
                    visual.Box.Size = Vector2.new(width, height)
                    visual.Box.Position = Vector2.new(rootPos.X - width/2, rootPos.Y - height/2)
                    visual.Box.Color = _G.ESPBoxColor
                    visual.Box.Visible = true
                    
                    visual.BoxOutline.Size = visual.Box.Size
                    visual.BoxOutline.Position = visual.Box.Position
                    visual.BoxOutline.Visible = true
                else
                    visual.Box.Visible = false
                    visual.BoxOutline.Visible = false
                end
                
                if _G.ESPNames then
                    visual.Name.Position = Vector2.new(rootPos.X, rootPos.Y - (math.abs(headPos.Y - legPos.Y) / 2) - 15)
                    visual.Name.Text = player.Name
                    visual.Name.Color = _G.ESPNameColor
                    visual.Name.Visible = true
                else
                    visual.Name.Visible = false
                end
                
                if _G.ESPDistance then
                    local distance = math.floor((root.Position - Camera.CFrame.Position).Magnitude)
                    visual.Distance.Position = Vector2.new(rootPos.X, rootPos.Y + (math.abs(headPos.Y - legPos.Y) / 2) + 5)
                    visual.Distance.Text = tostring(distance) .. "m"
                    visual.Distance.Color = _G.ESPNameColor
                    visual.Distance.Visible = true
                else
                    visual.Distance.Visible = false
                end
                
                if _G.ESPHealthBar then
                    local height = math.abs(headPos.Y - legPos.Y)
                    local width = height / 2
                    local healthPercent = humanoid.Health / humanoid.MaxHealth
                    
                    visual.HealthBar.Size = Vector2.new(2, height * healthPercent)
                    visual.HealthBar.Position = Vector2.new(rootPos.X - width/2 - 5, rootPos.Y - height/2 + (height * (1 - healthPercent)))
                    visual.HealthBar.Color = Color3.fromRGB(255 * (1 - healthPercent), 255 * healthPercent, 0)
                    visual.HealthBar.Visible = true
                    
                    visual.HealthBarOutline.Size = Vector2.new(2, height)
                    visual.HealthBarOutline.Position = Vector2.new(rootPos.X - width/2 - 5, rootPos.Y - height/2)
                    visual.HealthBarOutline.Visible = true
                else
                    visual.HealthBar.Visible = false
                    visual.HealthBarOutline.Visible = false
                end
                
                if _G.ESPTracers then
                    visual.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    visual.Tracer.To = Vector2.new(rootPos.X, rootPos.Y)
                    visual.Tracer.Color = _G.TracerColor
                    visual.Tracer.Visible = true
                else
                    visual.Tracer.Visible = false
                end
            else
                visual.Box.Visible = false
                visual.BoxOutline.Visible = false
                visual.Name.Visible = false
                visual.Distance.Visible = false
                visual.HealthBar.Visible = false
                visual.HealthBarOutline.Visible = false
                visual.Tracer.Visible = false
            end
        else
            visual.Box.Visible = false
            visual.BoxOutline.Visible = false
            visual.Name.Visible = false
            visual.Distance.Visible = false
            visual.HealthBar.Visible = false
            visual.HealthBarOutline.Visible = false
            visual.Tracer.Visible = false
        end
    end
end)

-- CRIAR UI CUSTOMIZADA
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PererelpsMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

if gethui then
    ScreenGui.Parent = gethui()
elseif syn and syn.protect_gui then
    syn.protect_gui(ScreenGui)
    ScreenGui.Parent = game.CoreGui
else
    ScreenGui.Parent = game.CoreGui
end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "Main"
MainFrame.Size = UDim2.new(0, 600, 0, 480)
MainFrame.Position = UDim2.new(0.5, -300, 0.5, -240)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 18)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 18)
UICorner.Parent = MainFrame

-- Gradient Background
local BackgroundGradient = Instance.new("UIGradient")
BackgroundGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(12, 12, 18)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(18, 15, 25)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(12, 12, 18))
}
BackgroundGradient.Rotation = 135
BackgroundGradient.Parent = MainFrame

-- Animated Background Pattern
local Pattern = Instance.new("ImageLabel")
Pattern.Name = "Pattern"
Pattern.Size = UDim2.new(1, 0, 1, 0)
Pattern.BackgroundTransparency = 1
Pattern.Image = "rbxassetid://8992230677"
Pattern.ImageColor3 = Color3.fromRGB(255, 0, 150)
Pattern.ImageTransparency = 0.95
Pattern.ScaleType = Enum.ScaleType.Tile
Pattern.TileSize = UDim2.new(0, 100, 0, 100)
Pattern.Parent = MainFrame

spawn(function()
    while wait(0.03) do
        if Pattern and Pattern.Parent then
            Pattern.TileSize = UDim2.new(0, 100 + math.sin(tick() * 2) * 10, 0, 100 + math.cos(tick() * 2) * 10)
        end
    end
end)

local Shadow = Instance.new("ImageLabel")
Shadow.Name = "Shadow"
Shadow.Size = UDim2.new(1, 50, 1, 50)
Shadow.Position = UDim2.new(0, -25, 0, -25)
Shadow.BackgroundTransparency = 1
Shadow.Image = "rbxassetid://1316045217"
Shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
Shadow.ImageTransparency = 0.3
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(10, 10, 118, 118)
Shadow.ZIndex = 0
Shadow.Parent = MainFrame

-- Outer Glow
local OuterGlow = Instance.new("ImageLabel")
OuterGlow.Name = "OuterGlow"
OuterGlow.Size = UDim2.new(1, 60, 1, 60)
OuterGlow.Position = UDim2.new(0, -30, 0, -30)
OuterGlow.BackgroundTransparency = 1
OuterGlow.Image = "rbxassetid://1316045217"
OuterGlow.ImageColor3 = Color3.fromRGB(255, 0, 150)
OuterGlow.ImageTransparency = 0.7
OuterGlow.ScaleType = Enum.ScaleType.Slice
OuterGlow.SliceCenter = Rect.new(10, 10, 118, 118)
OuterGlow.ZIndex = 0
OuterGlow.Parent = MainFrame

-- Animated Outer Glow
spawn(function()
    while wait() do
        if OuterGlow and OuterGlow.Parent then
            TweenService:Create(OuterGlow, TweenInfo.new(1.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
                ImageTransparency = 0.5
            }):Play()
            
            TweenService:Create(OuterGlow, TweenInfo.new(3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
                ImageColor3 = Color3.fromRGB(200, 0, 255)
            }):Play()
        end
        break
    end
end)

-- Border Effect
local BorderFrame = Instance.new("Frame")
BorderFrame.Size = UDim2.new(1, 2, 1, 2)
BorderFrame.Position = UDim2.new(0, -1, 0, -1)
BorderFrame.BackgroundTransparency = 1
BorderFrame.Parent = MainFrame

local BorderStroke = Instance.new("UIStroke")
BorderStroke.Color = Color3.fromRGB(255, 0, 150)
BorderStroke.Thickness = 2
BorderStroke.Transparency = 0.5
BorderStroke.Parent = MainFrame

local BorderGradient = Instance.new("UIGradient")
BorderGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 150)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(200, 0, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 150))
}
BorderGradient.Rotation = 0
BorderGradient.Parent = BorderStroke

-- Animated Border
spawn(function()
    while wait(0.05) do
        if BorderGradient and BorderGradient.Parent then
            BorderGradient.Rotation = (BorderGradient.Rotation + 2) % 360
        end
    end
end)

local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 60)
TopBar.BackgroundTransparency = 1
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

-- Top Bar Gradient Background
local TopBarBg = Instance.new("Frame")
TopBarBg.Size = UDim2.new(1, 0, 1, 0)
TopBarBg.BackgroundColor3 = Color3.fromRGB(255, 0, 150)
TopBarBg.BorderSizePixel = 0
TopBarBg.Parent = TopBar

local TopBarGradient = Instance.new("UIGradient")
TopBarGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 150)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(220, 0, 200)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 0, 255))
}
TopBarGradient.Rotation = 45
TopBarGradient.Parent = TopBarBg

local TopBarCorner = Instance.new("UICorner")
TopBarCorner.CornerRadius = UDim.new(0, 18)
TopBarCorner.Parent = TopBarBg

local TopBarBottom = Instance.new("Frame")
TopBarBottom.Size = UDim2.new(1, 0, 0, 18)
TopBarBottom.Position = UDim2.new(0, 0, 1, -18)
TopBarBottom.BackgroundColor3 = Color3.fromRGB(255, 0, 150)
TopBarBottom.BorderSizePixel = 0
TopBarBottom.Parent = TopBarBg

local TopBarBottomGradient = Instance.new("UIGradient")
TopBarBottomGradient.Color = TopBarGradient.Color
TopBarBottomGradient.Rotation = 45
TopBarBottomGradient.Parent = TopBarBottom

-- Shine Effect on TopBar
local Shine = Instance.new("Frame")
Shine.Size = UDim2.new(0, 100, 1, 0)
Shine.Position = UDim2.new(-0.2, 0, 0, 0)
Shine.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
Shine.BackgroundTransparency = 0.7
Shine.BorderSizePixel = 0
Shine.Rotation = -20
Shine.Parent = TopBarBg

local ShineGradient = Instance.new("UIGradient")
ShineGradient.Transparency = NumberSequence.new{
    NumberSequenceKeypoint.new(0, 1),
    NumberSequenceKeypoint.new(0.5, 0),
    NumberSequenceKeypoint.new(1, 1)
}
ShineGradient.Parent = Shine

-- Animated Shine
spawn(function()
    while wait() do
        if Shine and Shine.Parent then
            local tween = TweenService:Create(Shine, TweenInfo.new(3, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1), {
                Position = UDim2.new(1.2, 0, 0, 0)
            })
            tween:Play()
        end
        break
    end
end)

-- Icon Container
local IconContainer = Instance.new("Frame")
IconContainer.Size = UDim2.new(0, 50, 0, 50)
IconContainer.Position = UDim2.new(0, 10, 0.5, -25)
IconContainer.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
IconContainer.BackgroundTransparency = 0.85
IconContainer.BorderSizePixel = 0
IconContainer.Parent = TopBar

local IconContainerCorner = Instance.new("UICorner")
IconContainerCorner.CornerRadius = UDim.new(0, 12)
IconContainerCorner.Parent = IconContainer

local IconGlow = Instance.new("ImageLabel")
IconGlow.Size = UDim2.new(1, 20, 1, 20)
IconGlow.Position = UDim2.new(0, -10, 0, -10)
IconGlow.BackgroundTransparency = 1
IconGlow.Image = "rbxassetid://1316045217"
IconGlow.ImageColor3 = Color3.fromRGB(255, 255, 255)
IconGlow.ImageTransparency = 0.6
IconGlow.ScaleType = Enum.ScaleType.Slice
IconGlow.SliceCenter = Rect.new(10, 10, 118, 118)
IconGlow.Parent = IconContainer

local IconLabel = Instance.new("TextLabel")
IconLabel.Size = UDim2.new(1, 0, 1, 0)
IconLabel.BackgroundTransparency = 1
IconLabel.Text = "💎"
IconLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
IconLabel.TextSize = 28
IconLabel.Font = Enum.Font.GothamBold
IconLabel.Parent = IconContainer

-- Rotating Icon
spawn(function()
    while wait(0.03) do
        if IconLabel and IconLabel.Parent then
            IconLabel.Rotation = (IconLabel.Rotation + 0.5) % 360
        end
    end
end)

local TitleContainer = Instance.new("Frame")
TitleContainer.Size = UDim2.new(1, -170, 1, 0)
TitleContainer.Position = UDim2.new(0, 68, 0, 0)
TitleContainer.BackgroundTransparency = 1
TitleContainer.Parent = TopBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 8)
Title.BackgroundTransparency = 1
Title.Text = "PERERELPS VIP"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.TextStrokeTransparency = 0.8
Title.Parent = TitleContainer

local Subtitle = Instance.new("TextLabel")
Subtitle.Size = UDim2.new(1, 0, 0, 18)
Subtitle.Position = UDim2.new(0, 0, 0, 34)
Subtitle.BackgroundTransparency = 1
Subtitle.Text = "✨ Premium Edition v7.0 Ultra ✨"
Subtitle.TextColor3 = Color3.fromRGB(255, 255, 255)
Subtitle.TextTransparency = 0.2
Subtitle.TextSize = 11
Subtitle.Font = Enum.Font.GothamMedium
Subtitle.TextXAlignment = Enum.TextXAlignment.Left
Subtitle.Parent = TitleContainer

local CloseButton = Instance.new("TextButton")
CloseButton.Name = "Close"
CloseButton.Size = UDim2.new(0, 40, 0, 40)
CloseButton.Position = UDim2.new(1, -50, 0.5, -20)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 40, 80)
CloseButton.Text = "✕"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 20
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Parent = TopBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 10)
CloseCorner.Parent = CloseButton

local CloseGlow = Instance.new("ImageLabel")
CloseGlow.Size = UDim2.new(1, 15, 1, 15)
CloseGlow.Position = UDim2.new(0, -7.5, 0, -7.5)
CloseGlow.BackgroundTransparency = 1
CloseGlow.Image = "rbxassetid://1316045217"
CloseGlow.ImageColor3 = Color3.fromRGB(255, 40, 80)
CloseGlow.ImageTransparency = 0.6
CloseGlow.ScaleType = Enum.ScaleType.Slice
CloseGlow.SliceCenter = Rect.new(10, 10, 118, 118)
CloseGlow.Parent = CloseButton

CloseButton.MouseEnter:Connect(function()
    TweenService:Create(CloseButton, TweenInfo.new(0.2), {
        Size = UDim2.new(0, 45, 0, 45),
        BackgroundColor3 = Color3.fromRGB(255, 60, 100)
    }):Play()
    
    TweenService:Create(CloseGlow, TweenInfo.new(0.2), {
        ImageTransparency = 0.3
    }):Play()
end)

CloseButton.MouseLeave:Connect(function()
    TweenService:Create(CloseButton, TweenInfo.new(0.2), {
        Size = UDim2.new(0, 40, 0, 40),
        BackgroundColor3 = Color3.fromRGB(255, 40, 80)
    }):Play()
    
    TweenService:Create(CloseGlow, TweenInfo.new(0.2), {
        ImageTransparency = 0.6
    }):Play()
end)

local TabContainer = Instance.new("Frame")
TabContainer.Name = "TabContainer"
TabContainer.Size = UDim2.new(0, 150, 1, -75)
TabContainer.Position = UDim2.new(0, 12, 0, 68)
TabContainer.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
TabContainer.BorderSizePixel = 0
TabContainer.Parent = MainFrame

local TabBgGradient = Instance.new("UIGradient")
TabBgGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(18, 18, 26)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(22, 20, 30))
}
TabBgGradient.Rotation = 90
TabBgGradient.Parent = TabContainer

local TabContainerStroke = Instance.new("UIStroke")
TabContainerStroke.Color = Color3.fromRGB(255, 0, 150)
TabContainerStroke.Transparency = 0.75
TabContainerStroke.Thickness = 1.5
TabContainerStroke.Parent = TabContainer

local TabStrokeGradient = Instance.new("UIGradient")
TabStrokeGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 150)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(200, 0, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 150))
}
TabStrokeGradient.Parent = TabContainerStroke

local TabCorner = Instance.new("UICorner")
TabCorner.CornerRadius = UDim.new(0, 12)
TabCorner.Parent = TabContainer

local TabList = Instance.new("UIListLayout")
TabList.Padding = UDim.new(0, 10)
TabList.SortOrder = Enum.SortOrder.LayoutOrder
TabList.Parent = TabContainer

local TabPadding = Instance.new("UIPadding")
TabPadding.PaddingTop = UDim.new(0, 10)
TabPadding.PaddingBottom = UDim.new(0, 10)
TabPadding.PaddingLeft = UDim.new(0, 10)
TabPadding.PaddingRight = UDim.new(0, 10)
TabPadding.Parent = TabContainer

local ContentContainer = Instance.new("Frame")
ContentContainer.Name = "ContentContainer"
ContentContainer.Size = UDim2.new(1, -180, 1, -75)
ContentContainer.Position = UDim2.new(0, 172, 0, 68)
ContentContainer.BackgroundTransparency = 1
ContentContainer.Parent = MainFrame

-- Função para criar Tab
local currentTab = nil

local function CreateTab(name, icon)
    local TabButton = Instance.new("TextButton")
    TabButton.Name = name
    TabButton.Size = UDim2.new(1, 0, 0, 42)
    TabButton.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    TabButton.Text = ""
    TabButton.Parent = TabContainer
    
    local TabGradient = Instance.new("UIGradient")
    TabGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(25, 25, 35)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 40))
    }
    TabGradient.Rotation = 90
    TabGradient.Parent = TabButton
    
    local TabCorner = Instance.new("UICorner")
    TabCorner.CornerRadius = UDim.new(0, 8)
    TabCorner.Parent = TabButton
    
    local TabStroke = Instance.new("UIStroke")
    TabStroke.Color = Color3.fromRGB(255, 0, 150)
    TabStroke.Transparency = 1
    TabStroke.Thickness = 1.5
    TabStroke.Parent = TabButton
    
    local TabIcon = Instance.new("TextLabel")
    TabIcon.Size = UDim2.new(0, 25, 1, 0)
    TabIcon.Position = UDim2.new(0, 10, 0, 0)
    TabIcon.BackgroundTransparency = 1
    TabIcon.Text = icon
    TabIcon.TextColor3 = Color3.fromRGB(180, 180, 190)
    TabIcon.TextSize = 18
    TabIcon.Font = Enum.Font.GothamBold
    TabIcon.Parent = TabButton
    
    local TabLabel = Instance.new("TextLabel")
    TabLabel.Size = UDim2.new(1, -45, 1, 0)
    TabLabel.Position = UDim2.new(0, 40, 0, 0)
    TabLabel.BackgroundTransparency = 1
    TabLabel.Text = name
    TabLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
    TabLabel.TextSize = 13
    TabLabel.Font = Enum.Font.GothamSemibold
    TabLabel.TextXAlignment = Enum.TextXAlignment.Left
    TabLabel.Parent = TabButton
    
    local TabContent = Instance.new("ScrollingFrame")
    TabContent.Name = name .. "Content"
    TabContent.Size = UDim2.new(1, 0, 1, 0)
    TabContent.BackgroundTransparency = 1
    TabContent.BorderSizePixel = 0
    TabContent.ScrollBarThickness = 3
    TabContent.ScrollBarImageColor3 = Color3.fromRGB(255, 0, 150)
    TabContent.Visible = false
    TabContent.Parent = ContentContainer
    
    local ContentList = Instance.new("UIListLayout")
    ContentList.Padding = UDim.new(0, 10)
    ContentList.SortOrder = Enum.SortOrder.LayoutOrder
    ContentList.Parent = TabContent
    
    local ContentPadding = Instance.new("UIPadding")
    ContentPadding.PaddingTop = UDim.new(0, 5)
    ContentPadding.PaddingRight = UDim.new(0, 5)
    ContentPadding.Parent = TabContent
    
    -- Hover effects
    TabButton.MouseEnter:Connect(function()
        if TabContent.Visible then return end
        
        TweenService:Create(TabButton, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(30, 30, 40)
        }):Play()
        
        TweenService:Create(TabStroke, TweenInfo.new(0.2), {
            Transparency = 0.7
        }):Play()
        
        TweenService:Create(TabIcon, TweenInfo.new(0.2), {
            TextColor3 = Color3.fromRGB(255, 0, 150)
        }):Play()
        
        TweenService:Create(TabLabel, TweenInfo.new(0.2), {
            TextColor3 = Color3.fromRGB(220, 220, 230)
        }):Play()
    end)
    
    TabButton.MouseLeave:Connect(function()
        if TabContent.Visible then return end
        
        TweenService:Create(TabButton, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(25, 25, 35)
        }):Play()
        
        TweenService:Create(TabStroke, TweenInfo.new(0.2), {
            Transparency = 1
        }):Play()
        
        TweenService:Create(TabIcon, TweenInfo.new(0.2), {
            TextColor3 = Color3.fromRGB(180, 180, 190)
        }):Play()
        
        TweenService:Create(TabLabel, TweenInfo.new(0.2), {
            TextColor3 = Color3.fromRGB(180, 180, 190)
        }):Play()
    end)
    
    TabButton.MouseButton1Click:Connect(function()
        for _, tab in pairs(ContentContainer:GetChildren()) do
            tab.Visible = false
        end
        TabContent.Visible = true
        
        for _, btn in pairs(TabContainer:GetChildren()) do
            if btn:IsA("TextButton") then
                TweenService:Create(btn, TweenInfo.new(0.3), {
                    BackgroundColor3 = Color3.fromRGB(25, 25, 35)
                }):Play()
                
                local stroke = btn:FindFirstChildOfClass("UIStroke")
                if stroke then
                    TweenService:Create(stroke, TweenInfo.new(0.3), {
                        Transparency = 1
                    }):Play()
                end
                
                for _, child in pairs(btn:GetChildren()) do
                    if child:IsA("TextLabel") then
                        TweenService:Create(child, TweenInfo.new(0.3), {
                            TextColor3 = Color3.fromRGB(180, 180, 190)
                        }):Play()
                    end
                end
            end
        end
        
        TweenService:Create(TabButton, TweenInfo.new(0.3), {
            BackgroundColor3 = Color3.fromRGB(255, 0, 150)
        }):Play()
        
        TweenService:Create(TabStroke, TweenInfo.new(0.3), {
            Transparency = 0
        }):Play()
        
        TweenService:Create(TabIcon, TweenInfo.new(0.3), {
            TextColor3 = Color3.fromRGB(255, 255, 255)
        }):Play()
        
        TweenService:Create(TabLabel, TweenInfo.new(0.3), {
            TextColor3 = Color3.fromRGB(255, 255, 255)
        }):Play()
    end)
    
    if not currentTab then
        currentTab = TabContent
        TabContent.Visible = true
        TabButton.BackgroundColor3 = Color3.fromRGB(255, 0, 150)
        TabStroke.Transparency = 0
        TabIcon.TextColor3 = Color3.fromRGB(255, 255, 255)
        TabLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    end
    
    return TabContent
end

-- Função para criar Toggle
local function CreateToggle(parent, text, default, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Size = UDim2.new(1, -10, 0, 38)
    ToggleFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 30)
    ToggleFrame.BorderSizePixel = 0
    ToggleFrame.Parent = parent
    
    local ToggleGradient = Instance.new("UIGradient")
    ToggleGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(22, 22, 30)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(28, 28, 38))
    }
    ToggleGradient.Rotation = 90
    ToggleGradient.Parent = ToggleFrame
    
    local ToggleCorner = Instance.new("UICorner")
    ToggleCorner.CornerRadius = UDim.new(0, 8)
    ToggleCorner.Parent = ToggleFrame
    
    local ToggleStroke = Instance.new("UIStroke")
    ToggleStroke.Color = Color3.fromRGB(255, 0, 150)
    ToggleStroke.Transparency = 0.85
    ToggleStroke.Thickness = 1
    ToggleStroke.Parent = ToggleFrame
    
    local ToggleLabel = Instance.new("TextLabel")
    ToggleLabel.Size = UDim2.new(1, -65, 1, 0)
    ToggleLabel.Position = UDim2.new(0, 12, 0, 0)
    ToggleLabel.BackgroundTransparency = 1
    ToggleLabel.Text = text
    ToggleLabel.TextColor3 = Color3.fromRGB(220, 220, 230)
    ToggleLabel.TextSize = 13
    ToggleLabel.Font = Enum.Font.GothamSemibold
    ToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    ToggleLabel.Parent = ToggleFrame
    
    local ToggleButton = Instance.new("TextButton")
    ToggleButton.Size = UDim2.new(0, 46, 0, 24)
    ToggleButton.Position = UDim2.new(1, -53, 0.5, -12)
    ToggleButton.BackgroundColor3 = default and Color3.fromRGB(50, 200, 80) or Color3.fromRGB(50, 50, 60)
    ToggleButton.Text = ""
    ToggleButton.Parent = ToggleFrame
    
    local ToggleButtonCorner = Instance.new("UICorner")
    ToggleButtonCorner.CornerRadius = UDim.new(1, 0)
    ToggleButtonCorner.Parent = ToggleButton
    
    local ToggleCircle = Instance.new("Frame")
    ToggleCircle.Size = UDim2.new(0, 18, 0, 18)
    ToggleCircle.Position = default and UDim2.new(1, -21, 0.5, -9) or UDim2.new(0, 3, 0.5, -9)
    ToggleCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    ToggleCircle.BorderSizePixel = 0
    ToggleCircle.Parent = ToggleButton
    
    local CircleCorner = Instance.new("UICorner")
    CircleCorner.CornerRadius = UDim.new(1, 0)
    CircleCorner.Parent = ToggleCircle
    
    local CircleGlow = Instance.new("ImageLabel")
    CircleGlow.Size = UDim2.new(1, 10, 1, 10)
    CircleGlow.Position = UDim2.new(0, -5, 0, -5)
    CircleGlow.BackgroundTransparency = 1
    CircleGlow.Image = "rbxassetid://1316045217"
    CircleGlow.ImageColor3 = default and Color3.fromRGB(50, 200, 80) or Color3.fromRGB(50, 50, 60)
    CircleGlow.ImageTransparency = 0.5
    CircleGlow.ScaleType = Enum.ScaleType.Slice
    CircleGlow.SliceCenter = Rect.new(10, 10, 118, 118)
    CircleGlow.Parent = ToggleCircle
    
    local toggled = default
    
    ToggleButton.MouseButton1Click:Connect(function()
        toggled = not toggled
        callback(toggled)
        
        TweenService:Create(ToggleButton, TweenInfo.new(0.3), {
            BackgroundColor3 = toggled and Color3.fromRGB(50, 200, 80) or Color3.fromRGB(50, 50, 60)
        }):Play()
        
        TweenService:Create(ToggleCircle, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
            Position = toggled and UDim2.new(1, -21, 0.5, -9) or UDim2.new(0, 3, 0.5, -9)
        }):Play()
        
        TweenService:Create(CircleGlow, TweenInfo.new(0.3), {
            ImageColor3 = toggled and Color3.fromRGB(50, 200, 80) or Color3.fromRGB(50, 50, 60)
        }):Play()
        
        TweenService:Create(ToggleStroke, TweenInfo.new(0.3), {
            Transparency = toggled and 0.5 or 0.85
        }):Play()
    end)
    
    -- Hover effect
    ToggleFrame.MouseEnter:Connect(function()
        TweenService:Create(ToggleStroke, TweenInfo.new(0.2), {
            Transparency = 0.6
        }):Play()
    end)
    
    ToggleFrame.MouseLeave:Connect(function()
        TweenService:Create(ToggleStroke, TweenInfo.new(0.2), {
            Transparency = toggled and 0.5 or 0.85
        }):Play()
    end)
end

-- Função para criar Slider
local function CreateSlider(parent, text, min, max, default, callback)
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Size = UDim2.new(1, -10, 0, 58)
    SliderFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 30)
    SliderFrame.BorderSizePixel = 0
    SliderFrame.Parent = parent
    
    local SliderGradient = Instance.new("UIGradient")
    SliderGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(22, 22, 30)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(28, 28, 38))
    }
    SliderGradient.Rotation = 90
    SliderGradient.Parent = SliderFrame
    
    local SliderCorner = Instance.new("UICorner")
    SliderCorner.CornerRadius = UDim.new(0, 8)
    SliderCorner.Parent = SliderFrame
    
    local SliderStroke = Instance.new("UIStroke")
    SliderStroke.Color = Color3.fromRGB(255, 0, 150)
    SliderStroke.Transparency = 0.85
    SliderStroke.Thickness = 1
    SliderStroke.Parent = SliderFrame
    
    local SliderLabel = Instance.new("TextLabel")
    SliderLabel.Size = UDim2.new(1, -24, 0, 22)
    SliderLabel.Position = UDim2.new(0, 12, 0, 8)
    SliderLabel.BackgroundTransparency = 1
    SliderLabel.Text = text .. ": " .. default
    SliderLabel.TextColor3 = Color3.fromRGB(220, 220, 230)
    SliderLabel.TextSize = 13
    SliderLabel.Font = Enum.Font.GothamSemibold
    SliderLabel.TextXAlignment = Enum.TextXAlignment.Left
    SliderLabel.Parent = SliderFrame
    
    local SliderBar = Instance.new("Frame")
    SliderBar.Size = UDim2.new(1, -24, 0, 7)
    SliderBar.Position = UDim2.new(0, 12, 1, -18)
    SliderBar.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    SliderBar.BorderSizePixel = 0
    SliderBar.Parent = SliderFrame
    
    local BarCorner = Instance.new("UICorner")
    BarCorner.CornerRadius = UDim.new(1, 0)
    BarCorner.Parent = SliderBar
    
    local SliderFill = Instance.new("Frame")
    SliderFill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    SliderFill.BackgroundColor3 = Color3.fromRGB(255, 0, 150)
    SliderFill.BorderSizePixel = 0
    SliderFill.Parent = SliderBar
    
    local FillCorner = Instance.new("UICorner")
    FillCorner.CornerRadius = UDim.new(1, 0)
    FillCorner.Parent = SliderFill
    
    local FillGradient = Instance.new("UIGradient")
    FillGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 150)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 0, 255))
    }
    FillGradient.Parent = SliderFill
    
    local SliderDot = Instance.new("Frame")
    SliderDot.Size = UDim2.new(0, 15, 0, 15)
    SliderDot.Position = UDim2.new((default - min) / (max - min), -7.5, 0.5, -7.5)
    SliderDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    SliderDot.BorderSizePixel = 0
    SliderDot.Parent = SliderBar
    
    local DotCorner = Instance.new("UICorner")
    DotCorner.CornerRadius = UDim.new(1, 0)
    DotCorner.Parent = SliderDot
    
    local DotGlow = Instance.new("ImageLabel")
    DotGlow.Size = UDim2.new(1, 10, 1, 10)
    DotGlow.Position = UDim2.new(0, -5, 0, -5)
    DotGlow.BackgroundTransparency = 1
    DotGlow.Image = "rbxassetid://1316045217"
    DotGlow.ImageColor3 = Color3.fromRGB(255, 0, 150)
    DotGlow.ImageTransparency = 0.5
    DotGlow.ScaleType = Enum.ScaleType.Slice
    DotGlow.SliceCenter = Rect.new(10, 10, 118, 118)
    DotGlow.Parent = SliderDot
    
    local dragging = false
    
    SliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            TweenService:Create(SliderDot, TweenInfo.new(0.2), {
                Size = UDim2.new(0, 18, 0, 18)
            }):Play()
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            TweenService:Create(SliderDot, TweenInfo.new(0.2), {
                Size = UDim2.new(0, 15, 0, 15)
            }):Play()
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mouse = UserInputService:GetMouseLocation()
            local bar = SliderBar.AbsolutePosition.X
            local size = SliderBar.AbsoluteSize.X
            local value = math.clamp((mouse.X - bar) / size, 0, 1)
            local result = math.floor(min + (max - min) * value)
            
            TweenService:Create(SliderFill, TweenInfo.new(0.1), {
                Size = UDim2.new(value, 0, 1, 0)
            }):Play()
            
            TweenService:Create(SliderDot, TweenInfo.new(0.1), {
                Position = UDim2.new(value, -7.5, 0.5, -7.5)
            }):Play()
            
            SliderLabel.Text = text .. ": " .. result
            callback(result)
        end
    end)
    
    -- Hover effect
    SliderFrame.MouseEnter:Connect(function()
        TweenService:Create(SliderStroke, TweenInfo.new(0.2), {
            Transparency = 0.6
        }):Play()
    end)
    
    SliderFrame.MouseLeave:Connect(function()
        if not dragging then
            TweenService:Create(SliderStroke, TweenInfo.new(0.2), {
                Transparency = 0.85
            }):Play()
        end
    end)
end

-- Criar Tabs
local AimbotTab = CreateTab("AIMBOT", "🎯")
local ESPTab = CreateTab("ESP", "👁️")
local SettingsTab = CreateTab("CONFIG", "⚙️")

-- Aimbot Controls
CreateToggle(AimbotTab, "Ativar Aimbot", false, function(v) _G.AimbotEnabled = v end)
CreateToggle(AimbotTab, "Team Check", false, function(v) _G.TeamCheck = v end)
CreateSlider(AimbotTab, "FOV Radius", 10, 500, 150, function(v) _G.FovRadius = v end)
CreateSlider(AimbotTab, "Smoothness", 0, 100, 50, function(v) _G.AimbotSmooth = v / 100 end)
CreateToggle(AimbotTab, "Mostrar FOV Circle", true, function(v) _G.FovVisible = v end)

-- ESP Controls
CreateToggle(ESPTab, "Ativar ESP", false, function(v) _G.ESPEnabled = v end)
CreateToggle(ESPTab, "Boxes", false, function(v) _G.ESPBoxes = v end)
CreateToggle(ESPTab, "Names", false, function(v) _G.ESPNames = v end)
CreateToggle(ESPTab, "Distance", false, function(v) _G.ESPDistance = v end)
CreateToggle(ESPTab, "Health Bar", false, function(v) _G.ESPHealthBar = v end)
CreateToggle(ESPTab, "Tracers", false, function(v) _G.ESPTracers = v end)

-- Settings
local InfoLabel = Instance.new("TextLabel")
InfoLabel.Size = UDim2.new(1, -10, 0, 60)
InfoLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
InfoLabel.Text = "💎 PERERELPS VIP\n\nAimbot: Segure botão direito\nMenu: Tecla K para abrir/fechar"
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
InfoLabel.TextSize = 12
InfoLabel.Font = Enum.Font.Gotham
InfoLabel.TextWrapped = true
InfoLabel.Parent = SettingsTab

local InfoCorner = Instance.new("UICorner")
InfoCorner.CornerRadius = UDim.new(0, 6)
InfoCorner.Parent = InfoLabel

-- Dragging
local dragging = false
local dragInput, mousePos, framePos

TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        mousePos = input.Position
        framePos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

RunService.RenderStepped:Connect(function()
    if dragging and dragInput then
        local delta = dragInput.Position - mousePos
        MainFrame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
    end
end)

-- Close Button
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
    FOVCircle:Remove()
    for _, v in pairs(PlayerVisuals) do 
        for _, drawing in pairs(v) do
            drawing:Remove()
        end
    end
end)

-- Toggle Menu (K key)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
        MainFrame.Visible = not MainFrame.Visible
        
        -- Animação suave
        if MainFrame.Visible then
            MainFrame.BackgroundTransparency = 1
            TweenService:Create(MainFrame, TweenInfo.new(0.3), {
                BackgroundTransparency = 0
            }):Play()
            
            for _, child in pairs(MainFrame:GetDescendants()) do
                if child:IsA("TextLabel") or child:IsA("TextButton") then
                    child.TextTransparency = 1
                    TweenService:Create(child, TweenInfo.new(0.3), {
                        TextTransparency = 0
                    }):Play()
                elseif child:IsA("Frame") and child ~= MainFrame then
                    child.BackgroundTransparency = 1
                    TweenService:Create(child, TweenInfo.new(0.3), {
                        BackgroundTransparency = 0
                    }):Play()
                end
            end
        end
    end
end)

-- Notification
game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "✅ PERERELPS VIP";
    Text = "Menu carregado! Pressione K para abrir/fechar";
    Duration = 5;
})

end -- Fim da função loadMainMenu
