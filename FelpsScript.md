-- PHANTOM HUB - PROFESSIONAL MENU
-- Design baseado na imagem fornecida
-- Key: Secreta

wait(3)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

wait(1)

print("==========================================")
print("PHANTOM HUB - LOADING...")
print("==========================================")

-- KEY SYSTEM
local CORRECT_KEY = "HFDEGH-FJHKWA-32GHJD"

-- CONFIGURAÇÕES
local Config = {
    AimbotEnabled = false,
    TeamCheck = false,
    FovRadius = 150,
    FovVisible = true,
    AimbotSmooth = 0.5, -- Valor interno (0.0-1.0)
    ESPEnabled = false,
    ESPBoxes = false,
    ESPNames = false,
    ESPDistance = false,
    ESPHealthBar = false,
    ESPTracers = false,
    ESPSkeleton = false,
    HitboxEnabled = false,
    TPEnabled = false
}

-- Hitbox storage
local OriginalSizes = {}

-- TP System
local TPTarget = nil


-- KEY GUI
local KeyScreenGui = Instance.new("ScreenGui")
KeyScreenGui.Name = "PhantomKeySystem"
KeyScreenGui.ResetOnSpawn = false
KeyScreenGui.Parent = PlayerGui

local KeyFrame = Instance.new("Frame")
KeyFrame.Size = UDim2.new(0, 400, 0, 280)
KeyFrame.Position = UDim2.new(0.5, -200, 0.5, -140)
KeyFrame.BackgroundColor3 = Color3.fromRGB(15, 10, 25)
KeyFrame.BorderSizePixel = 0
KeyFrame.Parent = KeyScreenGui

local KeyCorner = Instance.new("UICorner")
KeyCorner.CornerRadius = UDim.new(0, 12)
KeyCorner.Parent = KeyFrame

local KeyTopBar = Instance.new("Frame")
KeyTopBar.Size = UDim2.new(1, 0, 0, 50)
KeyTopBar.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
KeyTopBar.BorderSizePixel = 0
KeyTopBar.Parent = KeyFrame

local KeyTopCorner = Instance.new("UICorner")
KeyTopCorner.CornerRadius = UDim.new(0, 12)
KeyTopCorner.Parent = KeyTopBar

local KeyTopBottom = Instance.new("Frame")
KeyTopBottom.Size = UDim2.new(1, 0, 0, 12)
KeyTopBottom.Position = UDim2.new(0, 0, 1, -12)
KeyTopBottom.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
KeyTopBottom.BorderSizePixel = 0
KeyTopBottom.Parent = KeyTopBar

local KeyTitle = Instance.new("TextLabel")
KeyTitle.Size = UDim2.new(1, 0, 1, 0)
KeyTitle.BackgroundTransparency = 1
KeyTitle.Text = "☠️ ACCESS RESTRICTED ☠️"
KeyTitle.TextColor3 = Color3.new(1, 1, 1)
KeyTitle.TextSize = 20
KeyTitle.Font = Enum.Font.GothamBold
KeyTitle.Parent = KeyTopBar

-- Phantom Hub Icon (SUBSTITUA O ID PELO SEU!)
-- Como obter seu ID: Veja o arquivo PHANTOM_HUB_ICON_SETUP.txt
-- 1. Faça upload da imagem em create.roblox.com
-- 2. Pegue o ID da URL
-- 3. Substitua o número abaixo
local PhantomIcon = Instance.new("ImageLabel")
PhantomIcon.Size = UDim2.new(0, 70, 0, 70)
PhantomIcon.Position = UDim2.new(0.5, -35, 0, 65)
PhantomIcon.BackgroundTransparency = 1
PhantomIcon.Image = "rbxassetid://89528230598819" -- ← MUDE ESTE NÚMERO!
PhantomIcon.Parent = KeyFrame

local KeySubtitle = Instance.new("TextLabel")
KeySubtitle.Size = UDim2.new(1, -40, 0, 30)
KeySubtitle.Position = UDim2.new(0, 20, 0, 145)
KeySubtitle.BackgroundTransparency = 1
KeySubtitle.Text = "💀 PHANTOM HUB 💀"
KeySubtitle.TextColor3 = Color3.fromRGB(160, 80, 240)
KeySubtitle.TextSize = 18
KeySubtitle.Font = Enum.Font.GothamBold
KeySubtitle.Parent = KeyFrame

local KeyInput = Instance.new("TextBox")
KeyInput.Size = UDim2.new(1, -40, 0, 40)
KeyInput.Position = UDim2.new(0, 20, 0, 190)
KeyInput.BackgroundColor3 = Color3.fromRGB(25, 15, 40)
KeyInput.BorderSizePixel = 1
KeyInput.BorderColor3 = Color3.fromRGB(130, 50, 200)
KeyInput.PlaceholderText = "Enter Key..."
KeyInput.Text = ""
KeyInput.TextColor3 = Color3.new(1, 1, 1)
KeyInput.PlaceholderColor3 = Color3.fromRGB(100, 50, 150)
KeyInput.TextSize = 14
KeyInput.Font = Enum.Font.Gotham
KeyInput.Parent = KeyFrame

local KeyInputCorner = Instance.new("UICorner")
KeyInputCorner.CornerRadius = UDim.new(0, 8)
KeyInputCorner.Parent = KeyInput

local SubmitButton = Instance.new("TextButton")
SubmitButton.Size = UDim2.new(1, -40, 0, 40)
SubmitButton.Position = UDim2.new(0, 20, 0, 235)
SubmitButton.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
SubmitButton.Text = "✓ VERIFY ACCESS"
SubmitButton.TextColor3 = Color3.new(1, 1, 1)
SubmitButton.TextSize = 15
SubmitButton.Font = Enum.Font.GothamBold
SubmitButton.Parent = KeyFrame

local SubmitCorner = Instance.new("UICorner")
SubmitCorner.CornerRadius = UDim.new(0, 8)
SubmitCorner.Parent = SubmitButton

-- ESP SYSTEM
local function CreateESP(player)
    local drawings = {
        Box = Drawing.new("Square"),
        BoxOutline = Drawing.new("Square"),
        Name = Drawing.new("Text"),
        Distance = Drawing.new("Text"),
        HealthBar = Drawing.new("Square"),
        HealthBarOutline = Drawing.new("Square"),
        Tracer = Drawing.new("Line"),
        HeadTorso = Drawing.new("Line"),
        TorsoLeftArm = Drawing.new("Line"),
        TorsoRightArm = Drawing.new("Line"),
        TorsoLeftLeg = Drawing.new("Line"),
        TorsoRightLeg = Drawing.new("Line")
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
    
    drawings.Distance.Size = 13
    drawings.Distance.Center = true
    drawings.Distance.Outline = true
    drawings.Distance.Visible = false
    
    drawings.HealthBar.Filled = true
    drawings.HealthBar.Visible = false
    
    drawings.HealthBarOutline.Filled = false
    drawings.HealthBarOutline.Color = Color3.fromRGB(0, 0, 0)
    drawings.HealthBarOutline.Visible = false
    
    drawings.Tracer.Thickness = 1.5
    drawings.Tracer.Visible = false
    
    -- Skeleton
    for i = 8, 12 do
        local line = drawings[({[8]="HeadTorso",[9]="TorsoLeftArm",[10]="TorsoRightArm",[11]="TorsoLeftLeg",[12]="TorsoRightLeg"})[i]]
        line.Thickness = 2
        line.Color = Color3.new(1, 1, 1)
        line.Visible = false
    end
    
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

-- Inicializar ESP para players existentes
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        PlayerVisuals[player] = CreateESP(player)
    end
end

local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Transparency = 1
FOVCircle.NumSides = 32

-- MAIN LOOP (OTIMIZADO - SEM LAG)
local lastHitboxUpdate = 0
local HITBOX_UPDATE_INTERVAL = 0.1 -- Atualizar hitbox a cada 0.1s em vez de todo frame

RunService:BindToRenderStep("FelpsUpdate", 200, function()
    pcall(function()
        -- FOV Circle
        if Config.AimbotEnabled and Config.FovVisible then
            FOVCircle.Radius = Config.FovRadius
            FOVCircle.Position = UserInputService:GetMouseLocation()
            FOVCircle.Color = Color3.fromRGB(130, 50, 200)
            FOVCircle.Visible = true
        else
            FOVCircle.Visible = false
        end

        -- Aimbot (COM SMOOTHNESS FUNCIONANDO)
        if Config.AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
            local target = nil
            local dist = Config.FovRadius
            
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
                    if Config.TeamCheck and p.Team == LocalPlayer.Team then continue end
                    
                    local pos, vis = Camera:WorldToViewportPoint(p.Character.Head.Position)
                    if vis then
                        local mag = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                        if mag < dist then 
                            target = p.Character.Head
                            dist = mag 
                        end
                    end
                end
            end
            
            if target then 
                -- Smoothness aplicado corretamente
                local targetCFrame = CFrame.lookAt(Camera.CFrame.Position, target.Position)
                Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, Config.AimbotSmooth)
            end
        end
        
        -- Hitbox (OTIMIZADO - atualiza a cada 0.1s)
        local currentTime = tick()
        if currentTime - lastHitboxUpdate >= HITBOX_UPDATE_INTERVAL then
            lastHitboxUpdate = currentTime
            
            if Config.HitboxEnabled then
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character then
                        local head = player.Character:FindFirstChild("Head")
                        if head then
                            if not OriginalSizes[head] then
                                OriginalSizes[head] = head.Size
                            end
                            
                            if not (Config.TeamCheck and player.Team == LocalPlayer.Team) then
                                head.Size = Vector3.new(5, 5, 5)
                                head.Transparency = 0.5
                                head.CanCollide = false
                            end
                        end
                    end
                end
            else
                -- Restaurar hitboxes
                for h, originalSize in pairs(OriginalSizes) do
                    if h and h.Parent then
                        h.Size = originalSize
                        h.Transparency = 0
                        h.CanCollide = true
                    end
                end
                OriginalSizes = {}
            end
        end
        
        -- TP System (teleportar para o player selecionado)
        if Config.TPEnabled and TPTarget then
            local targetPlayer = Players:FindFirstChild(TPTarget)
            if targetPlayer and targetPlayer.Character and LocalPlayer.Character then
                local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                local myRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                
                if targetRoot and myRoot then
                    myRoot.CFrame = targetRoot.CFrame * CFrame.new(0, 0, TPDistance)
                end
            end
        end
        
        -- Auto Heal (loop forçado para manter HP em 100%)
        if Config.AutoHeal then
            if LocalPlayer.Character then
                local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.Health = humanoid.MaxHealth
                end
            end
        end

        -- ESP UPDATE (OTIMIZADO)
        if Config.ESPEnabled then
            for player, visual in pairs(PlayerVisuals) do
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and 
                   player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                    
                    local root = player.Character.HumanoidRootPart
                    local head = player.Character:FindFirstChild("Head")
                    local humanoid = player.Character.Humanoid
                    
                    if not head then continue end
                    
                    local rootPos, rootVis = Camera:WorldToViewportPoint(root.Position)
                    
                    if not rootVis then
                        for _, drawing in pairs(visual) do
                            drawing.Visible = false
                        end
                        continue
                    end
                    
                    local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
                    local legPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))
                    
                    local isEnemy = (not Config.TeamCheck or player.Team ~= LocalPlayer.Team)
                    
                    if isEnemy then
                        -- Boxes
                        if Config.ESPBoxes then
                            local height = math.abs(headPos.Y - legPos.Y)
                            local width = height / 2
                            
                            visual.Box.Size = Vector2.new(width, height)
                            visual.Box.Position = Vector2.new(rootPos.X - width/2, rootPos.Y - height/2)
                            visual.Box.Color = Color3.new(1, 1, 1)
                            visual.Box.Visible = true
                            
                            visual.BoxOutline.Size = visual.Box.Size
                            visual.BoxOutline.Position = visual.Box.Position
                            visual.BoxOutline.Visible = true
                        else
                            visual.Box.Visible = false
                            visual.BoxOutline.Visible = false
                        end
                        
                        -- Names
                        if Config.ESPNames then
                            visual.Name.Position = Vector2.new(rootPos.X, rootPos.Y - (math.abs(headPos.Y - legPos.Y) / 2) - 15)
                            visual.Name.Text = player.Name
                            visual.Name.Color = Color3.new(1, 1, 1)
                            visual.Name.Visible = true
                        else
                            visual.Name.Visible = false
                        end
                        
                        -- Distance
                        if Config.ESPDistance then
                            local distance = math.floor((root.Position - Camera.CFrame.Position).Magnitude)
                            visual.Distance.Position = Vector2.new(rootPos.X, rootPos.Y + (math.abs(headPos.Y - legPos.Y) / 2) + 5)
                            visual.Distance.Text = tostring(distance) .. "m"
                            visual.Distance.Color = Color3.new(1, 1, 1)
                            visual.Distance.Visible = true
                        else
                            visual.Distance.Visible = false
                        end
                        
                        -- Health Bar
                        if Config.ESPHealthBar then
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
                        
                        -- Tracers
                        if Config.ESPTracers then
                            visual.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                            visual.Tracer.To = Vector2.new(rootPos.X, rootPos.Y)
                            visual.Tracer.Color = Color3.new(1, 1, 1)
                            visual.Tracer.Visible = true
                        else
                            visual.Tracer.Visible = false
                        end
                        
                        -- Skeleton
                        if Config.ESPSkeleton then
                            local larm = player.Character:FindFirstChild("Left Arm") or player.Character:FindFirstChild("LeftUpperArm")
                            local rarm = player.Character:FindFirstChild("Right Arm") or player.Character:FindFirstChild("RightUpperArm")
                            local lleg = player.Character:FindFirstChild("Left Leg") or player.Character:FindFirstChild("LeftUpperLeg")
                            local rleg = player.Character:FindFirstChild("Right Leg") or player.Character:FindFirstChild("RightUpperLeg")
                            
                            local hp = Camera:WorldToViewportPoint(head.Position)
                            local rp = Camera:WorldToViewportPoint(root.Position)
                            
                            visual.HeadTorso.From = Vector2.new(hp.X, hp.Y)
                            visual.HeadTorso.To = Vector2.new(rp.X, rp.Y)
                            visual.HeadTorso.Visible = true
                            
                            if larm then
                                local lap = Camera:WorldToViewportPoint(larm.Position)
                                visual.TorsoLeftArm.From = Vector2.new(rp.X, rp.Y)
                                visual.TorsoLeftArm.To = Vector2.new(lap.X, lap.Y)
                                visual.TorsoLeftArm.Visible = true
                            end
                            
                            if rarm then
                                local rap = Camera:WorldToViewportPoint(rarm.Position)
                                visual.TorsoRightArm.From = Vector2.new(rp.X, rp.Y)
                                visual.TorsoRightArm.To = Vector2.new(rap.X, rap.Y)
                                visual.TorsoRightArm.Visible = true
                            end
                            
                            if lleg then
                                local llp = Camera:WorldToViewportPoint(lleg.Position)
                                visual.TorsoLeftLeg.From = Vector2.new(rp.X, rp.Y)
                                visual.TorsoLeftLeg.To = Vector2.new(llp.X, llp.Y)
                                visual.TorsoLeftLeg.Visible = true
                            end
                            
                            if rleg then
                                local rlp = Camera:WorldToViewportPoint(rleg.Position)
                                visual.TorsoRightLeg.From = Vector2.new(rp.X, rp.Y)
                                visual.TorsoRightLeg.To = Vector2.new(rlp.X, rlp.Y)
                                visual.TorsoRightLeg.Visible = true
                            end
                        else
                            visual.HeadTorso.Visible = false
                            visual.TorsoLeftArm.Visible = false
                            visual.TorsoRightArm.Visible = false
                            visual.TorsoLeftLeg.Visible = false
                            visual.TorsoRightLeg.Visible = false
                        end
                    else
                        for _, drawing in pairs(visual) do
                            drawing.Visible = false
                        end
                    end
                else
                    for _, drawing in pairs(visual) do
                        drawing.Visible = false
                    end
                end
            end
        else
            -- Desativar todo ESP quando Config.ESPEnabled = false
            for _, visual in pairs(PlayerVisuals) do
                for _, drawing in pairs(visual) do
                    drawing.Visible = false
                end
            end
        end
    end)
end)

-- Validar Key
local function validateKey()
    if KeyInput.Text == CORRECT_KEY then
        TweenService:Create(SubmitButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(50, 255, 50)}):Play()
        SubmitButton.Text = "✓ CORRETO!"
        wait(0.5)
        KeyScreenGui:Destroy()
        loadMainMenu()
    else
        TweenService:Create(SubmitButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(220, 50, 50)}):Play()
        wait(0.3)
        TweenService:Create(SubmitButton, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(50, 200, 50)}):Play()
    end
end

SubmitButton.MouseButton1Click:Connect(validateKey)
KeyInput.FocusLost:Connect(function(enter) if enter then validateKey() end end)

-- CARREGAR MENU
function loadMainMenu()
    
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PhantomHub"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 750, 0, 500)
MainFrame.Position = UDim2.new(0.5, -375, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 28)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

-- Sombra do menu
local Shadow = Instance.new("ImageLabel")
Shadow.Size = UDim2.new(1, 30, 1, 30)
Shadow.Position = UDim2.new(0, -15, 0, -15)
Shadow.BackgroundTransparency = 1
Shadow.Image = "rbxassetid://89528230598819"
Shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
Shadow.ImageTransparency = 0.5
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(10, 10, 118, 118)
Shadow.ZIndex = 0
Shadow.Parent = MainFrame

-- Sidebar
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 60, 1, 0)
Sidebar.BackgroundColor3 = Color3.fromRGB(18, 18, 20)
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local SidebarCorner = Instance.new("UICorner")
SidebarCorner.CornerRadius = UDim.new(0, 10)
SidebarCorner.Parent = Sidebar

local SidebarCover = Instance.new("Frame")
SidebarCover.Size = UDim2.new(0, 10, 1, 0)
SidebarCover.Position = UDim2.new(1, -10, 0, 0)
SidebarCover.BackgroundColor3 = Color3.fromRGB(18, 18, 20)
SidebarCover.BorderSizePixel = 0
SidebarCover.Parent = Sidebar

-- Sidebar Logo (MESMO ID DO ÍCONE ACIMA!)
local Logo = Instance.new("ImageLabel")
Logo.Size = UDim2.new(1, 0, 0, 60)
Logo.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
Logo.Image = "rbxassetid://89528230598819" -- ← MUDE ESTE NÚMERO!
Logo.ScaleType = Enum.ScaleType.Fit
Logo.Parent = Sidebar

local LogoCorner = Instance.new("UICorner")
LogoCorner.CornerRadius = UDim.new(0, 10)
LogoCorner.Parent = Logo

local LogoCover = Instance.new("Frame")
LogoCover.Size = UDim2.new(0, 10, 0, 10)
LogoCover.Position = UDim2.new(1, -10, 1, -10)
LogoCover.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
LogoCover.BorderSizePixel = 0
LogoCover.Parent = Logo

-- Header
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, -60, 0, 50)
Header.Position = UDim2.new(0, 60, 0, 0)
Header.BackgroundColor3 = Color3.fromRGB(20, 20, 23)
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 20, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "PHANTOM HUB"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextSize = 18
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Header

-- Botões Header
local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 35, 0, 35)
MinBtn.Position = UDim2.new(1, -100, 0.5, -17.5)
MinBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 38)
MinBtn.Text = "−"
MinBtn.TextColor3 = Color3.new(1, 1, 1)
MinBtn.TextSize = 22
MinBtn.Font = Enum.Font.GothamBold
MinBtn.Parent = Header

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 6)
MinCorner.Parent = MinBtn

MinBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

local DestroyBtn = Instance.new("TextButton")
DestroyBtn.Size = UDim2.new(0, 35, 0, 35)
DestroyBtn.Position = UDim2.new(1, -55, 0.5, -17.5)
DestroyBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
DestroyBtn.Text = "×"
DestroyBtn.TextColor3 = Color3.new(1, 1, 1)
DestroyBtn.TextSize = 22
DestroyBtn.Font = Enum.Font.GothamBold
DestroyBtn.Parent = Header

local DestroyCorner = Instance.new("UICorner")
DestroyCorner.CornerRadius = UDim.new(0, 6)
DestroyCorner.Parent = DestroyBtn

DestroyBtn.MouseButton1Click:Connect(function()
    -- Desativar Auto Heal primeiro
    Config.AutoHeal = false
    pcall(DisableAutoHeal)
    
    -- Parar main loop
    pcall(function()
        RunService:UnbindFromRenderStep("FelpsUpdate")
    end)
    
    -- Desativar todas as configs
    Config.AimbotEnabled = false
    Config.TeamCheck = false
    Config.ESPEnabled = false
    Config.ESPBoxes = false
    Config.ESPNames = false
    Config.ESPDistance = false
    Config.ESPHealthBar = false
    Config.ESPTracers = false
    Config.ESPSkeleton = false
    Config.HitboxEnabled = false
    Config.TPEnabled = false
    
    -- Restaurar hitboxes
    for h, originalSize in pairs(OriginalSizes) do
        pcall(function()
            if h and h.Parent then
                h.Size = originalSize
                h.Transparency = 0
                h.CanCollide = true
            end
        end)
    end
    
    -- Remover FOV Circle
    pcall(function()
        if FOVCircle then
            FOVCircle.Visible = false
            FOVCircle:Remove()
        end
    end)
    
    -- Remover ESP
    for _, v in pairs(PlayerVisuals) do 
        for _, d in pairs(v) do 
            pcall(function()
                d.Visible = false
                d:Remove()
            end)
        end
    end
    
    print("☠️ Phantom Hub closing...")
    
    -- DESTRUIR TUDO IMEDIATAMENTE (sem wait)
    spawn(function()
        -- Destruir ScreenGui principal
        pcall(function()
            ScreenGui:Destroy()
        end)
        
        -- Destruir KeyScreenGui
        pcall(function()
            KeyScreenGui:Destroy()
        end)
        
        -- Limpeza extra - remover qualquer GUI relacionado
        task.wait(0.05)
        pcall(function()
            for _, gui in pairs(game:GetService("Players").LocalPlayer.PlayerGui:GetChildren()) do
                if gui.Name == "PhantomHub" or gui.Name == "PhantomKeySystem" or gui.Name == "FelpsMenu" then
                    gui:Destroy()
                end
            end
        end)
        
        print("✅ Phantom Hub completely destroyed!")
    end)
end)

-- Content
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -70, 1, -60)
Content.Position = UDim2.new(0, 65, 0, 55)
Content.BackgroundTransparency = 1
Content.Parent = MainFrame

-- Criar Section
local function Section(name, x, y, w, h)
    local s = Instance.new("Frame")
    s.Size = UDim2.new(0, w, 0, h)
    s.Position = UDim2.new(0, x, 0, y)
    s.BackgroundColor3 = Color3.fromRGB(30, 30, 33)
    s.BorderSizePixel = 0
    s.Parent = Content
    
    local sCorner = Instance.new("UICorner")
    sCorner.CornerRadius = UDim.new(0, 8)
    sCorner.Parent = s
    
    local h = Instance.new("Frame")
    h.Size = UDim2.new(1, 0, 0, 35)
    h.BackgroundColor3 = Color3.fromRGB(35, 35, 38)
    h.BorderSizePixel = 0
    h.Parent = s
    
    local hCorner = Instance.new("UICorner")
    hCorner.CornerRadius = UDim.new(0, 8)
    hCorner.Parent = h
    
    local hCover = Instance.new("Frame")
    hCover.Size = UDim2.new(1, 0, 0, 8)
    hCover.Position = UDim2.new(0, 0, 1, -8)
    hCover.BackgroundColor3 = Color3.fromRGB(35, 35, 38)
    hCover.BorderSizePixel = 0
    hCover.Parent = h
    
    local t = Instance.new("TextLabel")
    t.Size = UDim2.new(1, -15, 1, 0)
    t.Position = UDim2.new(0, 15, 0, 0)
    t.BackgroundTransparency = 1
    t.Text = name
    t.TextColor3 = Color3.new(1, 1, 1)
    t.TextSize = 13
    t.Font = Enum.Font.GothamBold
    t.TextXAlignment = Enum.TextXAlignment.Left
    t.Parent = h
    
    local d = Instance.new("Frame")
    d.Size = UDim2.new(1, 0, 0, 1)
    d.Position = UDim2.new(0, 0, 0, 35)
    d.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
    d.BorderSizePixel = 0
    d.Parent = s
    
    return s
end

-- Toggle
local function Toggle(p, n, k, y)
    local t = Instance.new("Frame")
    t.Size = UDim2.new(1, -20, 0, 35)
    t.Position = UDim2.new(0, 10, 0, y)
    t.BackgroundTransparency = 1
    t.Parent = p
    
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, -60, 1, 0)
    l.BackgroundTransparency = 1
    l.Text = n
    l.TextColor3 = Color3.fromRGB(200, 200, 200)
    l.TextSize = 12
    l.Font = Enum.Font.Gotham
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = t
    
    local sw = Instance.new("TextButton")
    sw.Size = UDim2.new(0, 40, 0, 20)
    sw.Position = UDim2.new(1, -45, 0.5, -10)
    sw.BackgroundColor3 = Color3.fromRGB(60, 60, 63)
    sw.Text = ""
    sw.Parent = t
    
    local swCorner = Instance.new("UICorner")
    swCorner.CornerRadius = UDim.new(1, 0)
    swCorner.Parent = sw
    
    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 16, 0, 16)
    dot.Position = UDim2.new(0, 2, 0.5, -8)
    dot.BackgroundColor3 = Color3.new(1, 1, 1)
    dot.BorderSizePixel = 0
    dot.Parent = sw
    
    local dotCorner = Instance.new("UICorner")
    dotCorner.CornerRadius = UDim.new(1, 0)
    dotCorner.Parent = dot
    
    sw.MouseButton1Click:Connect(function()
        Config[k] = not Config[k]
        
        if Config[k] then
            TweenService:Create(sw, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(130, 50, 200)}):Play()
            TweenService:Create(dot, TweenInfo.new(0.2), {Position = UDim2.new(1, -18, 0.5, -8)}):Play()
        else
            TweenService:Create(sw, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 63)}):Play()
            TweenService:Create(dot, TweenInfo.new(0.2), {Position = UDim2.new(0, 2, 0.5, -8)}):Play()
        end
    end)
end

-- Slider
local function Slider(p, n, k, min, max, y)
    local s = Instance.new("Frame")
    s.Size = UDim2.new(1, -20, 0, 50)
    s.Position = UDim2.new(0, 10, 0, y)
    s.BackgroundTransparency = 1
    s.Parent = p
    
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, -50, 0, 20)
    l.BackgroundTransparency = 1
    l.Text = n
    l.TextColor3 = Color3.fromRGB(200, 200, 200)
    l.TextSize = 11
    l.Font = Enum.Font.Gotham
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = s
    
    local displayValue = Config[k]
    if k == "AimbotSmooth" then
        displayValue = math.floor(Config[k] * 100)
    end
    
    local v = Instance.new("TextLabel")
    v.Size = UDim2.new(0, 40, 0, 20)
    v.Position = UDim2.new(1, -40, 0, 0)
    v.BackgroundTransparency = 1
    v.Text = tostring(displayValue)
    v.TextColor3 = Color3.fromRGB(130, 50, 200)
    v.TextSize = 11
    v.Font = Enum.Font.GothamBold
    v.TextXAlignment = Enum.TextXAlignment.Right
    v.Parent = s
    
    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1, 0, 0, 4)
    bar.Position = UDim2.new(0, 0, 0, 30)
    bar.BackgroundColor3 = Color3.fromRGB(45, 45, 48)
    bar.BorderSizePixel = 0
    bar.Parent = s
    
    local fillPct = (displayValue - min) / (max - min)
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(fillPct, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
    fill.BorderSizePixel = 0
    fill.Parent = bar
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.Parent = bar
    
    local drag = false
    btn.MouseButton1Down:Connect(function() drag = true end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end end)
    
    RunService.RenderStepped:Connect(function()
        if drag then
            local m = UserInputService:GetMouseLocation()
            local r = math.clamp(m.X - bar.AbsolutePosition.X, 0, bar.AbsoluteSize.X)
            local pct = r / bar.AbsoluteSize.X
            local val = math.floor(min + (max - min) * pct)
            
            if k == "AimbotSmooth" then
                Config[k] = val / 100
                v.Text = tostring(val)
            else
                Config[k] = val
                v.Text = tostring(val)
            end
            
            fill.Size = UDim2.new(pct, 0, 1, 0)
        end
    end)
end

-- BUILD
local s1 = Section("AIMBOT", 5, 5, 215, 265)
Toggle(s1, "Enabled (Hold RMB)", "AimbotEnabled", 45)
Toggle(s1, "Team Check", "TeamCheck", 85)
Slider(s1, "FOV Radius", "FovRadius", 10, 500, 125)
Slider(s1, "Smoothness", "AimbotSmooth", 1, 100, 185)

local aimbotNote = Instance.new("TextLabel")
aimbotNote.Size = UDim2.new(1, -20, 0, 30)
aimbotNote.Position = UDim2.new(0, 10, 0, 230)
aimbotNote.BackgroundTransparency = 1
aimbotNote.Text = "Higher = Smoother\nLower = Faster"
aimbotNote.TextColor3 = Color3.fromRGB(120, 120, 120)
aimbotNote.TextSize = 9
aimbotNote.Font = Enum.Font.Gotham
aimbotNote.TextYAlignment = Enum.TextYAlignment.Top
aimbotNote.Parent = s1

local s2 = Section("HITBOX", 225, 5, 215, 100)
Toggle(s2, "Hitbox Expander (5x5x5)", "HitboxEnabled", 45)

local s3 = Section("ESP", 445, 5, 215, 330)
Toggle(s3, "Enabled", "ESPEnabled", 45)
Toggle(s3, "Boxes", "ESPBoxes", 85)
Toggle(s3, "Names", "ESPNames", 125)
Toggle(s3, "Distance", "ESPDistance", 165)
Toggle(s3, "Health Bar", "ESPHealthBar", 205)
Toggle(s3, "Tracers", "ESPTracers", 245)
Toggle(s3, "Skeleton", "ESPSkeleton", 285)

-- TELEPORT Section (movida para não sobrepor)
local s4 = Section("TELEPORT", 225, 110, 215, 240)

-- TP System UI
local tpLabel = Instance.new("TextLabel")
tpLabel.Size = UDim2.new(1, -20, 0, 20)
tpLabel.Position = UDim2.new(0, 10, 0, 45)
tpLabel.BackgroundTransparency = 1
tpLabel.Text = "Select Player:"
tpLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
tpLabel.TextSize = 11
tpLabel.Font = Enum.Font.GothamBold
tpLabel.TextXAlignment = Enum.TextXAlignment.Left
tpLabel.Parent = s4

-- Container para dropdown e refresh button
local dropdownContainer = Instance.new("Frame")
dropdownContainer.Size = UDim2.new(1, -20, 0, 35)
dropdownContainer.Position = UDim2.new(0, 10, 0, 70)
dropdownContainer.BackgroundTransparency = 1
dropdownContainer.Parent = s4

-- Dropdown de players
local playerDropdown = Instance.new("TextButton")
playerDropdown.Size = UDim2.new(1, -45, 0, 35)
playerDropdown.Position = UDim2.new(0, 0, 0, 0)
playerDropdown.BackgroundColor3 = Color3.fromRGB(35, 35, 38)
playerDropdown.Text = "Select Player..."
playerDropdown.TextColor3 = Color3.fromRGB(200, 200, 200)
playerDropdown.TextSize = 11
playerDropdown.Font = Enum.Font.Gotham
playerDropdown.ZIndex = 5
playerDropdown.Parent = dropdownContainer

local dropdownCorner = Instance.new("UICorner")
dropdownCorner.CornerRadius = UDim.new(0, 6)
dropdownCorner.Parent = playerDropdown

-- Botão Refresh
local refreshBtn = Instance.new("TextButton")
refreshBtn.Size = UDim2.new(0, 35, 0, 35)
refreshBtn.Position = UDim2.new(1, -40, 0, 0)
refreshBtn.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
refreshBtn.Text = "🔄"
refreshBtn.TextColor3 = Color3.new(1, 1, 1)
refreshBtn.TextSize = 16
refreshBtn.Font = Enum.Font.GothamBold
refreshBtn.ZIndex = 5
refreshBtn.Parent = dropdownContainer

local refreshCorner = Instance.new("UICorner")
refreshCorner.CornerRadius = UDim.new(0, 6)
refreshCorner.Parent = refreshBtn

-- Lista de players (aparece FORA da section para não ser cortada)
local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(0, 170, 0, 120)
playerList.Position = UDim2.new(0, 235, 0, 215) -- Posição absoluta no Content
playerList.BackgroundColor3 = Color3.fromRGB(30, 30, 33)
playerList.BorderSizePixel = 1
playerList.BorderColor3 = Color3.fromRGB(130, 50, 200)
playerList.ScrollBarThickness = 4
playerList.Visible = false
playerList.ZIndex = 100
playerList.Parent = Content

local listCorner = Instance.new("UICorner")
listCorner.CornerRadius = UDim.new(0, 6)
listCorner.Parent = playerList

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 2)
listLayout.Parent = playerList

-- Função para atualizar lista de players
local function updatePlayerList()
    -- Limpar lista atual
    for _, child in pairs(playerList:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    -- Adicionar players
    local playerCount = 0
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            playerCount = playerCount + 1
            
            local playerBtn = Instance.new("TextButton")
            playerBtn.Size = UDim2.new(1, -10, 0, 28)
            playerBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 43)
            playerBtn.Text = player.Name
            playerBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
            playerBtn.TextSize = 10
            playerBtn.Font = Enum.Font.Gotham
            playerBtn.ZIndex = 101
            playerBtn.Parent = playerList
            
            local btnCorner = Instance.new("UICorner")
            btnCorner.CornerRadius = UDim.new(0, 4)
            btnCorner.Parent = playerBtn
            
            -- Hover effect
            playerBtn.MouseEnter:Connect(function()
                playerBtn.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
            end)
            
            playerBtn.MouseLeave:Connect(function()
                playerBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 43)
            end)
            
            playerBtn.MouseButton1Click:Connect(function()
                TPTarget = player.Name
                playerDropdown.Text = player.Name
                playerList.Visible = false
            end)
        end
    end
    
    if playerCount == 0 then
        local noPlayers = Instance.new("TextLabel")
        noPlayers.Size = UDim2.new(1, 0, 0, 30)
        noPlayers.BackgroundTransparency = 1
        noPlayers.Text = "No players found"
        noPlayers.TextColor3 = Color3.fromRGB(150, 150, 150)
        noPlayers.TextSize = 10
        noPlayers.Font = Enum.Font.Gotham
        noPlayers.ZIndex = 101
        noPlayers.Parent = playerList
    end
    
    playerList.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 5)
end

-- Toggle dropdown
playerDropdown.MouseButton1Click:Connect(function()
    playerList.Visible = not playerList.Visible
    if playerList.Visible then
        updatePlayerList()
    end
end)

-- Refresh button
refreshBtn.MouseButton1Click:Connect(function()
    updatePlayerList()
    
    -- Animação de feedback
    TweenService:Create(refreshBtn, TweenInfo.new(0.1), {
        Rotation = 180
    }):Play()
    
    wait(0.1)
    
    TweenService:Create(refreshBtn, TweenInfo.new(0.1), {
        Rotation = 0
    }):Play()
end)

-- Botão TP único
local tpButton = Instance.new("TextButton")
tpButton.Size = UDim2.new(1, -20, 0, 35)
tpButton.Position = UDim2.new(0, 10, 0, 115)
tpButton.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
tpButton.Text = "TELEPORT NOW"
tpButton.TextColor3 = Color3.new(1, 1, 1)
tpButton.TextSize = 12
tpButton.Font = Enum.Font.GothamBold
tpButton.Parent = s4

local tpBtnCorner = Instance.new("UICorner")
tpBtnCorner.CornerRadius = UDim.new(0, 6)
tpBtnCorner.Parent = tpButton

-- Container para Auto TP
local autoTPContainer = Instance.new("Frame")
autoTPContainer.Size = UDim2.new(1, -20, 0, 35)
autoTPContainer.Position = UDim2.new(0, 10, 0, 160)
autoTPContainer.BackgroundTransparency = 1
autoTPContainer.Parent = s4

local autoTPLabel = Instance.new("TextLabel")
autoTPLabel.Size = UDim2.new(1, -50, 1, 0)
autoTPLabel.BackgroundTransparency = 1
autoTPLabel.Text = "Auto TP (Loop)"
autoTPLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
autoTPLabel.TextSize = 11
autoTPLabel.Font = Enum.Font.Gotham
autoTPLabel.TextXAlignment = Enum.TextXAlignment.Left
autoTPLabel.Parent = autoTPContainer

-- Toggle TP contínuo
local tpToggle = Instance.new("TextButton")
tpToggle.Size = UDim2.new(0, 40, 0, 20)
tpToggle.Position = UDim2.new(1, -45, 0.5, -10)
tpToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 63)
tpToggle.Text = ""
tpToggle.Parent = autoTPContainer

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(1, 0)
toggleCorner.Parent = tpToggle

local toggleDot = Instance.new("Frame")
toggleDot.Size = UDim2.new(0, 16, 0, 16)
toggleDot.Position = UDim2.new(0, 2, 0.5, -8)
toggleDot.BackgroundColor3 = Color3.new(1, 1, 1)
toggleDot.BorderSizePixel = 0
toggleDot.Parent = tpToggle

local dotCorner2 = Instance.new("UICorner")
dotCorner2.CornerRadius = UDim.new(1, 0)
dotCorner2.Parent = toggleDot

-- TP único (botão)
tpButton.MouseButton1Click:Connect(function()
    if TPTarget then
        local targetPlayer = Players:FindFirstChild(TPTarget)
        if targetPlayer and targetPlayer.Character and LocalPlayer.Character then
            local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
            local myRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if targetRoot and myRoot then
                myRoot.CFrame = targetRoot.CFrame * CFrame.new(0, 0, TPDistance)
                
                -- Feedback visual
                TweenService:Create(tpButton, TweenInfo.new(0.1), {
                    BackgroundColor3 = Color3.fromRGB(50, 255, 100)
                }):Play()
                
                wait(0.1)
                
                TweenService:Create(tpButton, TweenInfo.new(0.1), {
                    BackgroundColor3 = Color3.fromRGB(130, 50, 200)
                }):Play()
            end
        end
    else
        -- Nenhum player selecionado
        TweenService:Create(tpButton, TweenInfo.new(0.1), {
            BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        }):Play()
        
        wait(0.2)
        
        TweenService:Create(tpButton, TweenInfo.new(0.1), {
            BackgroundColor3 = Color3.fromRGB(130, 50, 200)
        }):Play()
    end
end)

-- Toggle TP contínuo
tpToggle.MouseButton1Click:Connect(function()
    Config.TPEnabled = not Config.TPEnabled
    
    if Config.TPEnabled then
        TweenService:Create(tpToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(130, 50, 200)}):Play()
        TweenService:Create(toggleDot, TweenInfo.new(0.2), {Position = UDim2.new(1, -18, 0.5, -8)}):Play()
    else
        TweenService:Create(tpToggle, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 63)}):Play()
        TweenService:Create(toggleDot, TweenInfo.new(0.2), {Position = UDim2.new(0, 2, 0.5, -8)}):Play()
    end
end)

-- TP Distance Slider
local TPDistance = 3 -- Distância padrão

local distSliderFrame = Instance.new("Frame")
distSliderFrame.Size = UDim2.new(1, -20, 0, 45)
distSliderFrame.Position = UDim2.new(0, 10, 0, 205)
distSliderFrame.BackgroundTransparency = 1
distSliderFrame.Parent = s4

local distLabel = Instance.new("TextLabel")
distLabel.Size = UDim2.new(1, -50, 0, 18)
distLabel.BackgroundTransparency = 1
distLabel.Text = "TP Distance"
distLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
distLabel.TextSize = 10
distLabel.Font = Enum.Font.Gotham
distLabel.TextXAlignment = Enum.TextXAlignment.Left
distLabel.Parent = distSliderFrame

local distValue = Instance.new("TextLabel")
distValue.Size = UDim2.new(0, 35, 0, 18)
distValue.Position = UDim2.new(1, -35, 0, 0)
distValue.BackgroundTransparency = 1
distValue.Text = "3"
distValue.TextColor3 = Color3.fromRGB(130, 50, 200)
distValue.TextSize = 10
distValue.Font = Enum.Font.GothamBold
distValue.TextXAlignment = Enum.TextXAlignment.Right
distValue.Parent = distSliderFrame

local distBar = Instance.new("Frame")
distBar.Size = UDim2.new(1, 0, 0, 4)
distBar.Position = UDim2.new(0, 0, 0, 27)
distBar.BackgroundColor3 = Color3.fromRGB(45, 45, 48)
distBar.BorderSizePixel = 0
distBar.Parent = distSliderFrame

local distFill = Instance.new("Frame")
distFill.Size = UDim2.new((3-1)/(20-1), 0, 1, 0)
distFill.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
distFill.BorderSizePixel = 0
distFill.Parent = distBar

local distBtn = Instance.new("TextButton")
distBtn.Size = UDim2.new(1, 0, 1, 0)
distBtn.BackgroundTransparency = 1
distBtn.Text = ""
distBtn.Parent = distBar

local distDrag = false
distBtn.MouseButton1Down:Connect(function() distDrag = true end)
UserInputService.InputEnded:Connect(function(i) 
    if i.UserInputType == Enum.UserInputType.MouseButton1 then 
        distDrag = false 
    end 
end)

RunService.RenderStepped:Connect(function()
    if distDrag then
        local m = UserInputService:GetMouseLocation()
        local r = math.clamp(m.X - distBar.AbsolutePosition.X, 0, distBar.AbsoluteSize.X)
        local pct = r / distBar.AbsoluteSize.X
        local val = math.floor(1 + (20 - 1) * pct)
        
        TPDistance = val
        distValue.Text = tostring(val)
        distFill.Size = UDim2.new(pct, 0, 1, 0)
    end
end)

-- Quick TP Buttons
local quickTPLabel = Instance.new("TextLabel")
quickTPLabel.Size = UDim2.new(1, -20, 0, 15)
quickTPLabel.Position = UDim2.new(0, 10, 0, 255)
quickTPLabel.BackgroundTransparency = 1
quickTPLabel.Text = "Quick TP:"
quickTPLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
quickTPLabel.TextSize = 9
quickTPLabel.Font = Enum.Font.Gotham
quickTPLabel.TextXAlignment = Enum.TextXAlignment.Left
quickTPLabel.Parent = s4

-- Container for 3 buttons
local quickBtnsContainer = Instance.new("Frame")
quickBtnsContainer.Size = UDim2.new(1, -20, 0, 25)
quickBtnsContainer.Position = UDim2.new(0, 10, 0, 270)
quickBtnsContainer.BackgroundTransparency = 1
quickBtnsContainer.Parent = s4

local function createQuickTPBtn(text, pos, offsetFunc)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.32, -2, 1, 0)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
    btn.Text = text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextSize = 9
    btn.Font = Enum.Font.GothamBold
    btn.Parent = quickBtnsContainer
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = btn
    
    btn.MouseButton1Click:Connect(function()
        if TPTarget then
            local targetPlayer = Players:FindFirstChild(TPTarget)
            if targetPlayer and targetPlayer.Character and LocalPlayer.Character then
                local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                local myRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                
                if targetRoot and myRoot then
                    myRoot.CFrame = targetRoot.CFrame * offsetFunc() -- Use função para pegar valor atual
                    
                    TweenService:Create(btn, TweenInfo.new(0.1), {
                        BackgroundColor3 = Color3.fromRGB(130, 50, 200)
                    }):Play()
                    wait(0.15)
                    TweenService:Create(btn, TweenInfo.new(0.1), {
                        BackgroundColor3 = Color3.fromRGB(50, 50, 55)
                    }):Play()
                end
            end
        end
    end)
    
    return btn
end

createQuickTPBtn("BEHIND", UDim2.new(0, 0, 0, 0), function() return CFrame.new(0, 0, TPDistance) end)
createQuickTPBtn("FRONT", UDim2.new(0.34, 0, 0, 0), function() return CFrame.new(0, 0, -TPDistance) end)
createQuickTPBtn("ABOVE", UDim2.new(0.68, 0, 0, 0), function() return CFrame.new(0, TPDistance, 0) end)


-- Inicializar lista
updatePlayerList()

-- Fechar dropdown ao clicar fora
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        -- Fechar dropdown de TP
        if playerList.Visible then
            local mousePos = UserInputService:GetMouseLocation()
            local listPos = playerList.AbsolutePosition
            local listSize = playerList.AbsoluteSize
            
            -- Checar se clicou fora do dropdown e do botão
            if mousePos.X < listPos.X or mousePos.X > listPos.X + listSize.X or
               mousePos.Y < listPos.Y or mousePos.Y > listPos.Y + listSize.Y then
                
                local btnPos = playerDropdown.AbsolutePosition
                local btnSize = playerDropdown.AbsoluteSize
                
                if mousePos.X < btnPos.X or mousePos.X > btnPos.X + btnSize.X or
                   mousePos.Y < btnPos.Y or mousePos.Y > btnPos.Y + btnSize.Y then
                    playerList.Visible = false
                end
            end
        end
    end
end)

-- AUTO HEAL SECTION
local s5 = Section("AUTO HEAL", 5, 280, 215, 145)

-- Auto Heal Config
Config.AutoHeal = false
local autoHealConnection = nil

-- Função para ativar Auto Heal
local function EnableAutoHeal()
    if LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid and not autoHealConnection then
            autoHealConnection = humanoid.HealthChanged:Connect(function(health)
                if Config.AutoHeal and health < humanoid.MaxHealth then
                    humanoid.Health = humanoid.MaxHealth
                end
            end)
        end
    end
end

-- Função para desativar Auto Heal
local function DisableAutoHeal()
    if autoHealConnection then
        autoHealConnection:Disconnect()
        autoHealConnection = nil
    end
end

-- Reconectar Auto Heal quando morrer
LocalPlayer.CharacterAdded:Connect(function()
    wait(0.5)
    DisableAutoHeal()
    if Config.AutoHeal then
        EnableAutoHeal()
    end
end)

-- Custom Auto Heal Toggle with Enable/Disable functions
local autoHealToggleFrame = Instance.new("Frame")
autoHealToggleFrame.Size = UDim2.new(1, -20, 0, 35)
autoHealToggleFrame.Position = UDim2.new(0, 10, 0, 45)
autoHealToggleFrame.BackgroundTransparency = 1
autoHealToggleFrame.Parent = s5

local autoHealLabel = Instance.new("TextLabel")
autoHealLabel.Size = UDim2.new(1, -60, 1, 0)
autoHealLabel.BackgroundTransparency = 1
autoHealLabel.Text = "Auto Heal Enabled"
autoHealLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
autoHealLabel.TextSize = 12
autoHealLabel.Font = Enum.Font.Gotham
autoHealLabel.TextXAlignment = Enum.TextXAlignment.Left
autoHealLabel.Parent = autoHealToggleFrame

local autoHealSwitch = Instance.new("TextButton")
autoHealSwitch.Size = UDim2.new(0, 40, 0, 20)
autoHealSwitch.Position = UDim2.new(1, -45, 0.5, -10)
autoHealSwitch.BackgroundColor3 = Color3.fromRGB(60, 60, 63)
autoHealSwitch.Text = ""
autoHealSwitch.Parent = autoHealToggleFrame

local autoHealSwitchCorner = Instance.new("UICorner")
autoHealSwitchCorner.CornerRadius = UDim.new(1, 0)
autoHealSwitchCorner.Parent = autoHealSwitch

local autoHealDot = Instance.new("Frame")
autoHealDot.Size = UDim2.new(0, 16, 0, 16)
autoHealDot.Position = UDim2.new(0, 2, 0.5, -8)
autoHealDot.BackgroundColor3 = Color3.new(1, 1, 1)
autoHealDot.BorderSizePixel = 0
autoHealDot.Parent = autoHealSwitch

local autoHealDotCorner = Instance.new("UICorner")
autoHealDotCorner.CornerRadius = UDim.new(1, 0)
autoHealDotCorner.Parent = autoHealDot

autoHealSwitch.MouseButton1Click:Connect(function()
    Config.AutoHeal = not Config.AutoHeal
    
    if Config.AutoHeal then
        EnableAutoHeal()
        TweenService:Create(autoHealSwitch, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(130, 50, 200)}):Play()
        TweenService:Create(autoHealDot, TweenInfo.new(0.2), {Position = UDim2.new(1, -18, 0.5, -8)}):Play()
        
        game.StarterGui:SetCore("SendNotification", {
            Title = "💜 AUTO HEAL ON";
            Text = "HP stays at 100%";
            Duration = 2;
        })
    else
        DisableAutoHeal()
        TweenService:Create(autoHealSwitch, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 63)}):Play()
        TweenService:Create(autoHealDot, TweenInfo.new(0.2), {Position = UDim2.new(0, 2, 0.5, -8)}):Play()
        
        game.StarterGui:SetCore("SendNotification", {
            Title = "💜 AUTO HEAL OFF";
            Text = "Manual heal only";
            Duration = 2;
        })
    end
end)

-- Botão Heal Instantâneo
local healInstantButton = Instance.new("TextButton")
healInstantButton.Size = UDim2.new(1, -20, 0, 35)
healInstantButton.Position = UDim2.new(0, 10, 0, 90)
healInstantButton.BackgroundColor3 = Color3.fromRGB(130, 50, 200)
healInstantButton.Text = "💜 HEAL NOW"
healInstantButton.TextColor3 = Color3.new(1, 1, 1)
healInstantButton.TextSize = 12
healInstantButton.Font = Enum.Font.GothamBold
healInstantButton.Parent = s5

local healBtnCorner = Instance.new("UICorner")
healBtnCorner.CornerRadius = UDim.new(0, 6)
healBtnCorner.Parent = healInstantButton

healInstantButton.MouseButton1Click:Connect(function()
    if LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            local oldHealth = humanoid.Health
            for i = 1, 5 do
                humanoid.Health = humanoid.MaxHealth
                wait(0.05)
            end
            
            TweenService:Create(healInstantButton, TweenInfo.new(0.1), {
                BackgroundColor3 = Color3.fromRGB(160, 80, 240)
            }):Play()
            wait(0.2)
            TweenService:Create(healInstantButton, TweenInfo.new(0.2), {
                BackgroundColor3 = Color3.fromRGB(130, 50, 200)
            }):Play()
            
            game.StarterGui:SetCore("SendNotification", {
                Title = "💜 HEALED";
                Text = string.format("HP: %.0f → %.0f", oldHealth, humanoid.MaxHealth);
                Duration = 2;
            })
        end
    end
end)

local info = Instance.new("TextLabel")
info.Size = UDim2.new(0, 200, 0, 30)
info.Position = UDim2.new(0, 450, 1, -35)
info.BackgroundTransparency = 1
info.Text = "Press K to toggle"
info.TextColor3 = Color3.fromRGB(120, 120, 120)
info.TextSize = 11
info.Font = Enum.Font.Gotham
info.Parent = Content

UserInputService.InputBegan:Connect(function(i, g)
    if not g and i.KeyCode == Enum.KeyCode.K then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

game.StarterGui:SetCore("SendNotification", {
    Title = "☠️ PHANTOM HUB";
    Text = "Loaded! Press K";
    Duration = 5;
})

end

print("Phantom Hub loaded! Key: Secreta")
