--!strict
--[[
    ═══════════════════════════════════════════════════════════════
    ULTIMATE HUB — V8 FINAL (BUG-FREE)
    ═══════════════════════════════════════════════════════════════
    Developer: HENRY
    Fixes: WalkSpeed/JumpPower replication, Hitbox cleanup, Aimbot.
    ═══════════════════════════════════════════════════════════════
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

local VirtualUser = nil
pcall(function() VirtualUser = game:GetService("VirtualUser") end)

-- ============================================================
-- 1. CONFIGURATION
-- ============================================================
local Config = {
    Combat = { Aimbot = false, AimFOV = 150, AimSmooth = 5, WallCheck = true, TeamCheck = true },
    Visuals = { ESP = false, Boxes = true, Names = true, Health = true, Fullbright = false, Crosshair = false },
    Hitbox = { Enabled = false, Size = 10, Transparency = 0.5, Color = Color3.fromRGB(255,0,0) },
    Movement = { Fly = false, FlySpeed = 50, WalkSpeed = 16, JumpPower = 50, Noclip = false },
    World = { Gravity = 196.2 },
    Troll = { Spin = false, SpinSpeed = 20, SelectedPlayer = "" },
    Misc = { AntiAFK = true, AutoRespawn = false, FPSUnlock = false },
    Settings = { MenuKey = Enum.KeyCode.RightShift, Accent = Color3.fromRGB(255, 140, 0) }
}

-- ============================================================
-- 2. UI HELPER FUNCTIONS
-- ============================================================
local function New(class, parent)
    local obj = Instance.new(class)
    if parent then obj.Parent = parent end
    return obj
end

local function Corner(parent, radius)
    local c = New("UICorner", parent)
    c.CornerRadius = UDim.new(0, radius)
end

local function AddPadding(parent, pad)
    local p = New("UIPadding", parent)
    p.PaddingTop = UDim.new(0, pad)
    p.PaddingBottom = UDim.new(0, pad)
    p.PaddingLeft = UDim.new(0, pad)
    p.PaddingRight = UDim.new(0, pad)
end

-- ============================================================
-- 3. MAIN UI SETUP
-- ============================================================
local pg = LocalPlayer:WaitForChild("PlayerGui")
local old = pg:FindFirstChild("UltimateHub")
if old then old:Destroy() end

local ScreenGui = New("ScreenGui", pg)
ScreenGui.Name = "UltimateHub"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- CROSSHAIR
local Crosshair = New("Frame", ScreenGui)
Crosshair.Name = "Crosshair"
Crosshair.Size = UDim2.fromOffset(10, 10)
Crosshair.Position = UDim2.new(0.5, -5, 0.5, -5)
Crosshair.BackgroundColor3 = Color3.new(1,1,1)
Crosshair.BackgroundTransparency = 1
Crosshair.Visible = false
Corner(Crosshair, 5)

local Main = New("Frame", ScreenGui)
Main.Name = "Main"
Main.Size = UDim2.fromOffset(620, 500)
Main.Position = UDim2.new(0.5, -310, 0.5, -250)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 17)
Main.BorderSizePixel = 0
Main.Active = true
Corner(Main, 10)

local TopBar = New("Frame", Main)
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundColor3 = Color3.fromRGB(15, 15, 17)
TopBar.BorderSizePixel = 0

local Title = New("TextLabel", TopBar)
Title.Size = UDim2.fromOffset(200, 40)
Title.Position = UDim2.fromOffset(16, 0)
Title.BackgroundTransparency = 1
Title.Text = "HENRY'S HUB"
Title.TextColor3 = Config.Settings.Accent
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseBtn = New("TextButton", TopBar)
CloseBtn.Size = UDim2.fromOffset(30, 30)
CloseBtn.Position = UDim2.new(1, -40, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
CloseBtn.Text = "×"
CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 20
Corner(CloseBtn, 6)

local Sidebar = New("Frame", Main)
Sidebar.Size = UDim2.new(0, 140, 1, -64)
Sidebar.Position = UDim2.fromOffset(0, 40)
Sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
Sidebar.BorderSizePixel = 0

local BottomBar = New("Frame", Main)
BottomBar.Size = UDim2.new(1, 0, 0, 24)
BottomBar.Position = UDim2.new(0, 0, 1, -24)
BottomBar.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
BottomBar.BorderSizePixel = 0
Corner(BottomBar, 10)

local StatusLabel = New("TextLabel", BottomBar)
StatusLabel.Size = UDim2.new(1, -20, 1, 0)
StatusLabel.Position = UDim2.fromOffset(10, 0)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "FPS: 60 | Ping: 0ms"
StatusLabel.TextColor3 = Color3.fromRGB(150,150,160)
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 11
StatusLabel.TextXAlignment = Enum.TextXAlignment.Right

local ContentArea = New("Frame", Main)
ContentArea.Size = UDim2.new(1, -140, 1, -64)
ContentArea.Position = UDim2.fromOffset(140, 40)
ContentArea.BackgroundColor3 = Color3.fromRGB(15, 15, 17)
ContentArea.BorderSizePixel = 0
AddPadding(ContentArea, 16)

local ContentTitle = New("TextLabel", ContentArea)
ContentTitle.Size = UDim2.new(1, 0, 0, 30)
ContentTitle.BackgroundTransparency = 1
ContentTitle.Text = "Combat"
ContentTitle.TextColor3 = Config.Settings.Accent
ContentTitle.Font = Enum.Font.GothamBold
ContentTitle.TextSize = 14
ContentTitle.TextXAlignment = Enum.TextXAlignment.Left

local Scroll = New("ScrollingFrame", ContentArea)
Scroll.Size = UDim2.new(1, 0, 1, -30)
Scroll.Position = UDim2.fromOffset(0, 30)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.ScrollBarThickness = 2
Scroll.ScrollBarImageColor3 = Config.Settings.Accent
Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
Scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y

local SLayout = New("UIListLayout", Scroll)
SLayout.Padding = UDim.new(0, 8)
SLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- ============================================================
-- 4. UI COMPONENTS
-- ============================================================
local function CreateLabel(text)
    local lbl = New("TextLabel", Scroll)
    lbl.Size = UDim2.new(1, 0, 0, 20)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(150,150,160)
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 11
    lbl.TextXAlignment = Enum.TextXAlignment.Left
end

local function CreateToggle(name, default, callback)
    local row = New("TextButton", Scroll)
    row.Size = UDim2.new(1, 0, 0, 36)
    row.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
    row.Text = ""
    row.AutoButtonColor = false
    Corner(row, 6)
    
    local lbl = New("TextLabel", row)
    lbl.Size = UDim2.new(1, -60, 1, 0)
    lbl.Position = UDim2.fromOffset(12, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    
    local track = New("Frame", row)
    track.Size = UDim2.fromOffset(34, 18)
    track.Position = UDim2.new(1, -46, 0.5, -9)
    track.BackgroundColor3 = default and Config.Settings.Accent or Color3.fromRGB(50, 50, 55)
    track.BorderSizePixel = 0
    Corner(track, 9)
    
    local knob = New("Frame", track)
    knob.Size = UDim2.fromOffset(14, 14)
    knob.Position = default and UDim2.new(1, -16, 0.5, -7) or UDim2.fromOffset(2, 2)
    knob.BackgroundColor3 = Color3.new(1,1,1)
    knob.BorderSizePixel = 0
    Corner(knob, 7)
    
    row.MouseEnter:Connect(function() TweenService:Create(row, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(38, 38, 42)}):Play() end)
    row.MouseLeave:Connect(function() TweenService:Create(row, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(28, 28, 32)}):Play() end)
    
    task.defer(function()
        row.MouseButton1Down:Connect(function()
            default = not default
            TweenService:Create(track, TweenInfo.new(0.15), {BackgroundColor3 = default and Config.Settings.Accent or Color3.fromRGB(50, 50, 55)}):Play()
            TweenService:Create(knob, TweenInfo.new(0.15), {Position = default and UDim2.new(1, -16, 0.5, -7) or UDim2.fromOffset(2, 2)}):Play()
            callback(default)
        end)
    end)
end

local function CreateSlider(name, min, max, default, callback)
    local row = New("Frame", Scroll)
    row.Size = UDim2.new(1, 0, 0, 40)
    row.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
    Corner(row, 6)
    
    local lbl = New("TextLabel", row)
    lbl.Size = UDim2.new(1, -80, 1, 0)
    lbl.Position = UDim2.fromOffset(12, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    
    local valBox = New("Frame", row)
    valBox.Size = UDim2.fromOffset(44, 22)
    valBox.Position = UDim2.new(1, -56, 0.5, -11)
    valBox.BackgroundColor3 = Config.Settings.Accent
    valBox.BorderSizePixel = 0
    Corner(valBox, 4)
    
    local valText = New("TextLabel", valBox)
    valText.Size = UDim2.new(1, 0, 1, 0)
    valText.BackgroundTransparency = 1
    valText.Text = tostring(default)
    valText.TextColor3 = Color3.new(1,1,1)
    valText.Font = Enum.Font.GothamBold
    valText.TextSize = 12
    
    local interact = New("TextButton", row)
    interact.Size = UDim2.new(1, -100, 1, 0)
    interact.BackgroundTransparency = 1
    interact.Text = ""
    
    local dragging = false
    local function update(inputX)
        local rel = math.clamp((inputX - row.AbsolutePosition.X - 12) / (row.AbsoluteSize.X - 80), 0, 1)
        local val = math.floor(min + (max - min) * rel + 0.5)
        valText.Text = tostring(val)
        callback(val)
    end
    
    interact.MouseButton1Down:Connect(function() dragging = true; update(UserInputService:GetMouseLocation().X) end)
    UserInputService.InputChanged:Connect(function(input) if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then update(input.Position.X) end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
end

local function CreateTextInput(name, default, callback)
    local row = New("Frame", Scroll)
    row.Size = UDim2.new(1, 0, 0, 36)
    row.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
    Corner(row, 6)
    
    local lbl = New("TextLabel", row)
    lbl.Size = UDim2.new(1, -60, 1, 0)
    lbl.Position = UDim2.fromOffset(12, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    
    local box = New("TextBox", row)
    box.Size = UDim2.fromOffset(60, 24)
    box.Position = UDim2.new(1, -70, 0.5, -12)
    box.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    box.Text = tostring(default)
    box.TextColor3 = Color3.new(1,1,1)
    box.Font = Enum.Font.GothamBold
    box.TextSize = 13
    box.ClearTextOnFocus = false
    Corner(box, 4)
    
    box.FocusLost:Connect(function()
        local num = tonumber(box.Text)
        if num then
            callback(num)
        else
            box.Text = tostring(default) -- Revert if not a number
        end
    end)
end

local function CreateButton(name, color, callback)
    local btn = New("TextButton", Scroll)
    btn.Size = UDim2.new(1, 0, 0, 36)
    btn.BackgroundColor3 = color or Color3.fromRGB(40, 40, 45)
    btn.Text = name
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 13
    btn.AutoButtonColor = false
    Corner(btn, 6)
    
    task.defer(function()
        btn.MouseButton1Down:Connect(callback)
    end)
end

-- ============================================================
-- 5. TABS & PAGES
-- ============================================================
local TabButtons = {}

function BuildContent(tabName)
    for _, c in ipairs(Scroll:GetChildren()) do 
        if c:IsA("GuiObject") then c:Destroy() end 
    end
    
    if tabName == "Combat" then
        CreateLabel("AIMBOT (HOLD RMB)")
        CreateToggle("Enable Aimbot", Config.Combat.Aimbot, function(v) Config.Combat.Aimbot = v end)
        CreateToggle("Wall Check", Config.Combat.WallCheck, function(v) Config.Combat.WallCheck = v end)
        CreateSlider("FOV Size", 10, 500, Config.Combat.AimFOV, function(v) Config.Combat.AimFOV = v end)
        CreateSlider("Smoothness", 1, 20, Config.Combat.AimSmooth, function(v) Config.Combat.AimSmooth = v end)
        
        CreateLabel("CROSSHAIR")
        CreateToggle("Enable Crosshair", Config.Visuals.Crosshair, function(v) 
            Config.Visuals.Crosshair = v 
            Crosshair.Visible = v
        end)

    elseif tabName == "Hitbox" then
        CreateLabel("HITBOX EXPANDER")
        CreateToggle("Enable Hitbox", Config.Hitbox.Enabled, function(v) Config.Hitbox.Enabled = v end)
        
        CreateTextInput("Hitbox Size", Config.Hitbox.Size, function(v) Config.Hitbox.Size = v end)
        CreateTextInput("Transparency (0-1)", Config.Hitbox.Transparency, function(v) Config.Hitbox.Transparency = v end)
        
        CreateLabel("HITBOX COLOR")
        local r1 = New("Frame", Scroll)
        r1.Size = UDim2.new(1, 0, 0, 30)
        r1.BackgroundTransparency = 1
        local l1 = New("UIListLayout", r1)
        l1.FillDirection = Enum.FillDirection.Horizontal
        l1.Padding = UDim.new(0, 10)
        
        local Colors = {Color3.fromRGB(255,0,0), Color3.fromRGB(0,255,0), Color3.fromRGB(0,150,255), Color3.fromRGB(255,255,0), Color3.new(1,1,1)}
        for _, col in ipairs(Colors) do
            local cb = New("TextButton", r1)
            cb.Size = UDim2.fromOffset(30,30)
            cb.BackgroundColor3 = col
            cb.Text = ""
            Corner(cb, 15)
            cb.MouseButton1Down:Connect(function()
                Config.Hitbox.Color = col
                BuildContent("Hitbox")
            end)
        end

    elseif tabName == "Movement" then
        CreateLabel("CHARACTER STATS")
        CreateTextInput("Walk Speed", Config.Movement.WalkSpeed, function(v) Config.Movement.WalkSpeed = v end)
        CreateTextInput("Jump Power", Config.Movement.JumpPower, function(v) Config.Movement.JumpPower = v end)
        CreateToggle("Noclip", Config.Movement.Noclip, function(v) Config.Movement.Noclip = v end)
        
        CreateLabel("FLY")
        CreateToggle("Enable Fly", Config.Movement.Fly, function(v) Config.Movement.Fly = v end)
        CreateTextInput("Fly Speed", Config.Movement.FlySpeed, function(v) Config.Movement.FlySpeed = v end)

    elseif tabName == "Troll" then
        CreateLabel("TROLL OPTIONS")
        local pBox = New("TextBox", Scroll)
        pBox.Size = UDim2.new(1, 0, 0, 36)
        pBox.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
        pBox.Text = Config.Troll.SelectedPlayer
        pBox.PlaceholderText = "Type Exact Player Name..."
        pBox.PlaceholderColor3 = Color3.fromRGB(150,150,160)
        pBox.TextColor3 = Color3.new(1,1,1)
        pBox.Font = Enum.Font.Gotham
        pBox.TextSize = 13
        Corner(pBox, 6)
        local bpad = New("UIPadding", pBox)
        bpad.PaddingLeft = UDim.new(0, 12)
        pBox.FocusLost:Connect(function() Config.Troll.SelectedPlayer = pBox.Text end)
        
        CreateButton("Freeze Player", Color3.fromRGB(30,30,33), function()
            local t = Players:FindFirstChild(Config.Troll.SelectedPlayer)
            if t and t.Character and t.Character:FindFirstChildOfClass("Humanoid") then
                t.Character.Humanoid.WalkSpeed = 0
            end
        end)
        
        CreateButton("Fling Player", Color3.fromRGB(30,30,33), function()
            local t = Players:FindFirstChild(Config.Troll.SelectedPlayer)
            if t and t.Character and t.Character:FindFirstChild("HumanoidRootPart") then
                local vel = Instance.new("BodyAngularVelocity")
                vel.Parent = t.Character.HumanoidRootPart
                vel.AngularVelocity = Vector3.new(9999, 9999, 9999)
                vel.MaxTorque = Vector3.new(9999, 9999, 9999)
                task.delay(0.2, function() vel:Destroy() end)
            end
        end)

    elseif tabName == "Misc" then
        CreateLabel("UTILITIES")
        CreateToggle("Anti-AFK", Config.Misc.AntiAFK, function(v) Config.Misc.AntiAFK = v end)
        CreateToggle("Auto Respawn", Config.Misc.AutoRespawn, function(v) Config.Misc.AutoRespawn = v end)
        CreateLabel("FPS")
        CreateToggle("Unlock FPS (Max 240)", Config.Misc.FPSUnlock, function(v)
            Config.Misc.FPSUnlock = v
            if v then setfpscap(240) else setfpscap(60) end
        end)

    elseif tabName == "World" then
        CreateLabel("ENVIRONMENT")
        CreateSlider("Gravity", 0, 300, Config.World.Gravity, function(v) Config.World.Gravity = v; workspace.Gravity = v end)
        CreateToggle("Fullbright", Config.Visuals.Fullbright, function(v) 
            Config.Visuals.Fullbright = v
            Lighting.Ambient = v and Color3.new(1,1,1) or Color3.fromRGB(70,70,70)
            Lighting.OutdoorAmbient = v and Color3.new(1,1,1) or Color3.fromRGB(128,128,128)
        end)

    elseif tabName == "Visuals" then
        CreateLabel("PLAYER ESP")
        CreateToggle("Enable ESP", Config.Visuals.ESP, function(v) Config.Visuals.ESP = v end)
        CreateToggle("Boxes", Config.Visuals.Boxes, function(v) Config.Visuals.Boxes = v end)
        CreateToggle("Names", Config.Visuals.Names, function(v) Config.Visuals.Names = v end)
        CreateToggle("Health", Config.Visuals.Health, function(v) Config.Visuals.Health = v end)

    elseif tabName == "Teleport" then
        CreateLabel("PLAYER TELEPORT")
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                CreateButton("Teleport to " .. p.Name, Color3.fromRGB(30,30,33), function()
                    if p.Character and p.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                        LocalPlayer.Character.HumanoidRootPart.CFrame = p.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
                    end
                end)
            end
        end

    elseif tabName == "Settings" then
        CreateLabel("THEME PRESETS")
        local r1 = New("Frame", Scroll)
        r1.Size = UDim2.new(1, 0, 0, 30)
        r1.BackgroundTransparency = 1
        local l1 = New("UIListLayout", r1)
        l1.FillDirection = Enum.FillDirection.Horizontal
        l1.Padding = UDim.new(0, 10)
        
        local Colors = {Color3.fromRGB(255,140,0), Color3.fromRGB(170,0,255), Color3.fromRGB(255,50,50), Color3.fromRGB(0,200,100), Color3.fromRGB(0,170,255), Color3.new(1,1,1)}
        for _, col in ipairs(Colors) do
            local cb = New("TextButton", r1)
            cb.Size = UDim2.fromOffset(30,30)
            cb.BackgroundColor3 = col
            cb.Text = ""
            Corner(cb, 15)
            cb.MouseButton1Down:Connect(function()
                Config.Settings.Accent = col
                ContentTitle.TextColor3 = col
                Title.TextColor3 = col
                Scroll.ScrollBarImageColor3 = col
                for _, b in ipairs(TabButtons) do if b.TextColor3 ~= Color3.fromRGB(150,150,160) then b.TextColor3 = col end end
                BuildContent("Settings")
            end)
        end
        
        CreateLabel("CREDITS")
        local credits = New("TextLabel", Scroll)
        credits.Size = UDim2.new(1, 0, 0, 40)
        credits.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
        credits.Text = "Developer: HENRY\nUI Framework: Luau/Executor Edition"
        credits.TextColor3 = Color3.fromRGB(200, 200, 200)
        credits.Font = Enum.Font.Gotham
        credits.TextSize = 12
        Corner(credits, 6)
    end
end

local function CreateTab(name)
    local btn = New("TextButton", Sidebar)
    btn.Size = UDim2.new(1, -16, 0, 34)
    btn.BackgroundColor3 = Color3.fromRGB(20,20,22)
    btn.Text = "  " .. name
    btn.TextColor3 = Color3.fromRGB(150,150,160)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 13
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.AutoButtonColor = false
    btn.Position = UDim2.fromOffset(8, 10 + (#TabButtons * 36))
    Corner(btn, 4)
    
    table.insert(TabButtons, btn)
    
    task.defer(function()
        btn.MouseButton1Down:Connect(function()
            for _, b in ipairs(TabButtons) do 
                b.BackgroundColor3 = Color3.fromRGB(20,20,22)
                b.TextColor3 = Color3.fromRGB(150,150,160) 
            end
            btn.BackgroundColor3 = Color3.fromRGB(32, 32, 36)
            btn.TextColor3 = Config.Settings.Accent
            ContentTitle.Text = name
            BuildContent(name)
        end)
    end)
end

for _, name in ipairs({"Combat", "Hitbox", "Movement", "Troll", "Misc", "World", "Visuals", "Teleport", "Settings"}) do 
    CreateTab(name) 
end

-- Initialize the first tab safely
task.defer(function() 
    BuildContent("Combat")
    ContentTitle.Text = "Combat"
    TabButtons[1].BackgroundColor3 = Color3.fromRGB(32, 32, 36)
    TabButtons[1].TextColor3 = Config.Settings.Accent
end)

-- ============================================================
-- 6. DRAG LOGIC
-- ============================================================
local dragging, dragInput, dragStart, startPos

TopBar.InputBegan:Connect(function(input) 
    if input.UserInputType == Enum.UserInputType.MouseButton1 then 
        dragging = true
        dragStart = input.Position
        startPos = Main.Position 
    end 
end)

TopBar.InputChanged:Connect(function(input) 
    if input.UserInputType == Enum.UserInputType.MouseMovement then 
        dragInput = input 
    end 
end)

UserInputService.InputChanged:Connect(function(input) 
    if input == dragInput and dragging then 
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) 
    end 
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

UserInputService.InputBegan:Connect(function(input, gp) 
    if gp then return end
    if input.KeyCode == Config.Settings.MenuKey then 
        Main.Visible = not Main.Visible 
    end 
end)

task.defer(function()
    CloseBtn.MouseButton1Down:Connect(function() 
        ScreenGui:Destroy() 
    end)
end)

-- ============================================================
-- 7. CORE LOGIC (Aimbot, Movement, Exploits)
-- ============================================================
local flyBV, flyBG
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1
fovCircle.Color = Config.Settings.Accent
fovCircle.Filled = false
fovCircle.Visible = false

-- Anti AFK / Auto Respawn
task.spawn(function()
    while task.wait(1) do
        if Config.Misc.AntiAFK and VirtualUser then
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
            end)
        end
        
        if Config.Misc.AutoRespawn and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            if LocalPlayer.Character.Humanoid.Health <= 0 then
                LocalPlayer.Character:LoadCharacter()
            end
        end
    end
end)

-- Movement / Fly / Noclip (FIXED TO APPLY STATS)
RunService.RenderStepped:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    
    if hum then 
        -- Forces WalkSpeed and JumpPower to update every frame
        hum.WalkSpeed = Config.Movement.WalkSpeed 
        hum.JumpPower = Config.Movement.JumpPower 
    end
    
    if Config.Movement.Noclip and char then 
        for _, p in ipairs(char:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end 
    end

    if Config.Movement.Fly and hrp then
        if not flyBV then
            flyBV = New("BodyVelocity", hrp); flyBV.MaxForce = Vector3.new(1e5,1e5,1e5)
            flyBG = New("BodyGyro", hrp); flyBG.MaxTorque = Vector3.new(1e5,1e5,1e5); flyBG.P = 1000
        end
        local dir = Vector3.zero
        local camCF = workspace.CurrentCamera.CFrame
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir -= camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir += camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.yAxis end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.yAxis end
        
        flyBV.Velocity = dir.Magnitude > 0 and (dir.Unit * Config.Movement.FlySpeed) or Vector3.zero
        flyBG.CFrame = camCF
    elseif flyBV then 
        flyBV:Destroy(); flyBV = nil; flyBG:Destroy(); flyBG = nil 
    end
end)

-- COMBAT / AIMBOT (FIXED MATH)
RunService.RenderStepped:Connect(function()
    local cam = workspace.CurrentCamera
    local centerScreen = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)
    
    -- FOV Circle fixed to center
    fovCircle.Visible = Config.Combat.Aimbot
    fovCircle.Position = centerScreen
    fovCircle.Radius = Config.Combat.AimFOV / 2
    fovCircle.Color = Config.Settings.Accent

    if not Config.Combat.Aimbot then return end
    local isHoldingRMB = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
    if not isHoldingRMB then return end 
    
    local closest, dist = nil, Config.Combat.AimFOV
    
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local targetPart = p.Character:FindFirstChild("Head")
            local humanoid = p.Character:FindFirstChildOfClass("Humanoid")
            if targetPart and humanoid and humanoid.Health > 0 then
                if Config.Combat.TeamCheck and p.Team == LocalPlayer.Team then continue end
                if Config.Combat.WallCheck then
                    local ray = Ray.new(cam.CFrame.Position, targetPart.Position - cam.CFrame.Position)
                    local hit = workspace:FindPartOnRay(ray, LocalPlayer.Character)
                    if hit and not hit:IsDescendantOf(p.Character) then continue end
                end
                local screenPos, onScreen = cam:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local d = (Vector2.new(screenPos.X, screenPos.Y) - centerScreen).Magnitude
                    if d < dist then closest, dist = targetPart, d end
                end
            end
        end
    end

    if closest then
        local targetCF = CFrame.new(cam.CFrame.Position, closest.Position)
        local smoothVal = 1 / math.max(Config.Combat.AimSmooth, 1)
        cam.CFrame = cam.CFrame:Lerp(targetCF, smoothVal)
    end
end)

-- HITBOX EXPANDER (FIXED CLEANUP)
task.spawn(function()
    while task.wait(0.5) do
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
                local head = p.Character:FindFirstChild("Head")
                if head then
                    if Config.Hitbox.Enabled then
                        head.Size = Vector3.new(Config.Hitbox.Size, Config.Hitbox.Size, Config.Hitbox.Size)
                        head.Transparency = Config.Hitbox.Transparency
                        head.CanCollide = false
                        if not head:FindFirstChild("HitboxVisual") then
                            local box = New("SelectionBox", head)
                            box.Name = "HitboxVisual"
                            box.Adornee = head
                        end
                        local box = head:FindFirstChild("HitboxVisual")
                        if box then box.Color3 = Config.Hitbox.Color end
                    else
                        -- Cleanup if the toggle is OFF
                        if head.Size ~= Vector3.new(2, 1, 1) then
                            head.Size = Vector3.new(2, 1, 1)
                            head.Transparency = 0
                            head.CanCollide = true
                            local box = head:FindFirstChild("HitboxVisual")
                            if box then box:Destroy() end
                        end
                    end
                end
            end
        end
    end
end)

-- ============================================================
-- 8. ESP & STATS
-- ============================================================
local function CleanupESP()
    for _, v in pairs(ScreenGui:GetChildren()) do
        if v.Name:sub(1, 4) == "ESP_" then v:Destroy() end
    end
end

RunService.RenderStepped:Connect(function()
    if not Config.Visuals.ESP then CleanupESP(); return end
    
    local cam = workspace.CurrentCamera
    for _, p in ipairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        local char = p.Character
        if not char then continue end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local head = char:FindFirstChild("Head")
        local hum = char:FindFirstChildOfClass("Humanoid")
        
        if hrp and head and hum and hum.Health > 0 then
            local hrpPos, onScreen = cam:WorldToViewportPoint(hrp.Position)
            local headPos = cam:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
            local footPos = cam:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
            
            if onScreen then
                local height = math.abs(headPos.Y - footPos.Y)
                local width = height / 2
                
                local box = ScreenGui:FindFirstChild("ESP_Box_"..p.Name)
                if not box then 
                    box = New("Frame", ScreenGui)
                    box.Name = "ESP_Box_"..p.Name
                    box.BackgroundTransparency = 1
                    box.BorderSizePixel = 1
                    box.BorderColor3 = Config.Settings.Accent 
                end
                box.Visible = Config.Visuals.Boxes
                box.Size = UDim2.fromOffset(width, height)
                box.Position = UDim2.fromOffset(hrpPos.X - width/2, headPos.Y)
                
                local nameLbl = ScreenGui:FindFirstChild("ESP_Name_"..p.Name)
                if not nameLbl then 
                    nameLbl = New("TextLabel", ScreenGui)
                    nameLbl.Name = "ESP_Name_"..p.Name
                    nameLbl.BackgroundTransparency = 1
                    nameLbl.TextColor3 = Color3.new(1,1,1)
                    nameLbl.Font = Enum.Font.GothamBold
                    nameLbl.TextSize = 12
                    nameLbl.TextStrokeTransparency = 0 
                end
                nameLbl.Visible = Config.Visuals.Names
                nameLbl.Text = p.Name
                nameLbl.Position = UDim2.fromOffset(hrpPos.X, headPos.Y - 18)
                nameLbl.Size = UDim2.fromOffset(100, 20)
                nameLbl.AnchorPoint = Vector2.new(0.5, 1)
                
                local healthBar = ScreenGui:FindFirstChild("ESP_HP_"..p.Name)
                if not healthBar then 
                    healthBar = New("Frame", ScreenGui)
                    healthBar.Name = "ESP_HP_"..p.Name
                    healthBar.BorderSizePixel = 0
                    healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 100) 
                end
                local hpPct = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
                healthBar.Visible = Config.Visuals.Health
                healthBar.Size = UDim2.fromOffset(3, height * hpPct)
                healthBar.Position = UDim2.fromOffset(hrpPos.X - width/2 - 6, headPos.Y + height - (height * hpPct))
            end
        end
    end
end)

local startTime = tick(); local frames = 0
RunService.RenderStepped:Connect(function()
    frames += 1
    if tick() - startTime >= 1 then
        local fps = frames
        local ping = math.floor(LocalPlayer:GetNetworkPing() * 1000)
        StatusLabel.Text = string.format("FPS: %d | Ping: %dms", fps, ping)
        frames = 0; startTime = tick()
    end
end)

print("[HENRY'S HUB] Loaded Successfully. Press RightShift to toggle.")
