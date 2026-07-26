local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- Prevent duplicate GUIs
if CoreGui:FindFirstChild("XellwareHub") then
    CoreGui.XellwareHub:Destroy()
end

-- ==== VARIABLES ====
local HitboxEnabled = false
local HitboxSize = 2.0
local HitboxTrans = 0 
local GroundHitEnabled = false

local AutoWeaveEnabled = false
local CurrentWeaveState = false
local WeaveBlinkSpeed = 0.05
local AttackPauseUntil = 0
local NextWeaveToggle = 0

-- Locked internally: This guarantees auto-connect forces the hit to register 100%
local HitRegisterPauseTime = 0.15 

-- ==== RETRO UI THEME HELPER ====
local function applyRetroTheme(element)
    element.BackgroundColor3 = Color3.new(0, 0, 0)
    
    local stroke = Instance.new("UIStroke", element)
    stroke.Color = Color3.new(1, 1, 1)
    stroke.Thickness = 1
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    
    if element:IsA("TextLabel") or element:IsA("TextButton") or element:IsA("TextBox") then
        element.TextColor3 = Color3.new(1, 1, 1)
        element.Font = Enum.Font.Code 
    end
end

-- ==== UI CONSTRUCTION (TABBED) ====
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XellwareHub"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local OpenButton = Instance.new("TextButton", ScreenGui)
OpenButton.Position = UDim2.new(0, 10, 0, 10)
OpenButton.Size = UDim2.new(0, 60, 0, 40)
OpenButton.Text = "Open"
OpenButton.TextSize = 16
OpenButton.Visible = false
applyRetroTheme(OpenButton)

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -150)
MainFrame.Size = UDim2.new(0, 350, 0, 300)
MainFrame.Active = true
MainFrame.Draggable = true 
applyRetroTheme(MainFrame)

-- Top Bar
local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 35)
applyRetroTheme(TopBar)

local Title = Instance.new("TextLabel", TopBar)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 10, 0, 0)
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Text = "Xellware Hub - God Mode"
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.Code

local CloseButton = Instance.new("TextButton", TopBar)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Text = "X"
applyRetroTheme(CloseButton)

-- Tabs
local TabBar = Instance.new("Frame", MainFrame)
TabBar.Position = UDim2.new(0, 0, 0, 40)
TabBar.Size = UDim2.new(1, 0, 0, 35)
applyRetroTheme(TabBar)

local function createTabButton(text, posOffset)
    local btn = Instance.new("TextButton", TabBar)
    btn.Position = UDim2.new(posOffset, 5, 0, 0)
    btn.Size = UDim2.new(0.45, -5, 1, 0) 
    btn.Text = text
    btn.TextSize = 14
    applyRetroTheme(btn)
    return btn
end

local TabBtn_Hitbox = createTabButton("Hitbox", 0)
local TabBtn_Weave = createTabButton("God Weave", 0.5)

-- Pages
local PagesContainer = Instance.new("Frame", MainFrame)
PagesContainer.BackgroundTransparency = 1
PagesContainer.Position = UDim2.new(0, 10, 0, 85)
PagesContainer.Size = UDim2.new(1, -20, 1, -95)

local function createPage(name)
    local page = Instance.new("ScrollingFrame", PagesContainer)
    page.Name = name
    page.BackgroundTransparency = 1
    page.Size = UDim2.new(1, 0, 1, 0)
    page.ScrollBarThickness = 4
    page.Visible = false
    local layout = Instance.new("UIListLayout", page)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 8)
    return page
end

local Page_Hitbox = createPage("HitboxPage")
local Page_Weave = createPage("WeavePage")
Page_Weave.Visible = true 

-- Generators
local function createToggle(parent, text, defaultState)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, -10, 0, 35)
    btn.Text = text .. (defaultState and ": ON" or ": OFF")
    btn.TextSize = 16
    applyRetroTheme(btn)
    return btn
end

local function createSlider(parent, text, minVal, maxVal, defaultVal, callback)
    local container = Instance.new("Frame", parent)
    container.Size = UDim2.new(1, -10, 0, 45)
    applyRetroTheme(container)
    
    local label = Instance.new("TextLabel", container)
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Position = UDim2.new(0, 5, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. tostring(defaultVal)
    label.TextSize = 14
    label.Font = Enum.Font.Code
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local track = Instance.new("Frame", container)
    track.Size = UDim2.new(1, -20, 0, 10)
    track.Position = UDim2.new(0, 10, 0, 25)
    track.BackgroundColor3 = Color3.new(0, 0, 0)
    local stroke = Instance.new("UIStroke", track)
    stroke.Color = Color3.new(1, 1, 1)
    stroke.Thickness = 1
    
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.new(1, 1, 1)
    fill.BorderSizePixel = 0
    
    local button = Instance.new("TextButton", track)
    button.Size = UDim2.new(1, 0, 1, 0)
    button.BackgroundTransparency = 1
    button.Text = ""
    
    local dragging = false
    
    local function update(input)
        local pos = math.clamp((input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        local value = minVal + ((maxVal - minVal) * pos)
        value = math.floor(value * 100) / 100 
        label.Text = text .. ": " .. tostring(value)
        callback(value)
    end
    
    button.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            update(input)
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            update(input)
        end
    end)
    
    return container
end

-- Populate UI Elements
local Toggle_Hitbox = createToggle(Page_Hitbox, "Outlined Hitbox", false)
createSlider(Page_Hitbox, "Hitbox Size", 1.0, 6.5, 2.0, function(val) HitboxSize = val end)
createSlider(Page_Hitbox, "Outline Trans", 0.0, 1.0, 0.0, function(val) HitboxTrans = val end)
local Toggle_Ground = createToggle(Page_Hitbox, "Ground Hit (Ragdoll)", false)

local Toggle_Weave = createToggle(Page_Weave, "God Auto Weave", false)
createSlider(Page_Weave, "Blink Speed", 0.01, 0.30, 0.05, function(val) WeaveBlinkSpeed = val end)

-- ==== UI LOGIC ====
local function switchTab(activeBtn, activePage)
    Page_Hitbox.Visible, Page_Weave.Visible = false, false
    activePage.Visible = true
end

TabBtn_Hitbox.MouseButton1Click:Connect(function() switchTab(TabBtn_Hitbox, Page_Hitbox) end)
TabBtn_Weave.MouseButton1Click:Connect(function() switchTab(TabBtn_Weave, Page_Weave) end)

CloseButton.MouseButton1Click:Connect(function() MainFrame.Visible = false; OpenButton.Visible = true end)
OpenButton.MouseButton1Click:Connect(function() MainFrame.Visible = true; OpenButton.Visible = false end)

Toggle_Hitbox.MouseButton1Click:Connect(function() 
    HitboxEnabled = not HitboxEnabled
    Toggle_Hitbox.Text = "Outlined Hitbox: " .. (HitboxEnabled and "ON" or "OFF")
end)

Toggle_Ground.MouseButton1Click:Connect(function() 
    GroundHitEnabled = not GroundHitEnabled
    Toggle_Ground.Text = "Ground Hit (Ragdoll): " .. (GroundHitEnabled and "ON" or "OFF")
end)

Toggle_Weave.MouseButton1Click:Connect(function() 
    AutoWeaveEnabled = not AutoWeaveEnabled
    Toggle_Weave.Text = "God Auto Weave: " .. (AutoWeaveEnabled and "ON" or "OFF")
    
    if not AutoWeaveEnabled and LocalPlayer.Character then
        local core = LocalPlayer.Character:FindFirstChild("Core")
        if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
            core.Communicate[""]:FireServer("Weave", nil, false, nil)
            CurrentWeaveState = false
        end
    end
end)

-- ==== AUTO-CONNECT: FORCING HITS TO REGISTER ====
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            
            -- Establish window to ensure your damage connects 
            AttackPauseUntil = os.clock() + HitRegisterPauseTime
            
            -- Immediately unblink so the server reads the punch
            if AutoWeaveEnabled and CurrentWeaveState and LocalPlayer.Character then
                local core = LocalPlayer.Character:FindFirstChild("Core")
                if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                    core.Communicate[""]:FireServer("Weave", nil, false, nil)
                    CurrentWeaveState = false
                end
            end
        end
    end
end)

-- ==== GOD WEAVE BLINK LOOP ====
RunService.Heartbeat:Connect(function()
    if AutoWeaveEnabled and LocalPlayer.Character then
        local now = os.clock()
        
        -- If we are in the attack pause window, DO NOT BLINK. Keep weave off so hits land.
        if now < AttackPauseUntil then
            return 
        end
        
        if now >= NextWeaveToggle then
            local core = LocalPlayer.Character:FindFirstChild("Core")
            if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                local remote = core.Communicate[""]
                
                CurrentWeaveState = not CurrentWeaveState
                if CurrentWeaveState then
                    remote:FireServer("Weave", nil, true)
                else
                    remote:FireServer("Weave", nil, false, nil)
                end
                
                NextWeaveToggle = now + WeaveBlinkSpeed
            end
        end
    end
end)

-- ==== CORE HITBOX LOGIC (REWOUND) ====
RunService.RenderStepped:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            
            local hrp = player.Character.HumanoidRootPart
            local humanoid = player.Character.Humanoid
            local state = humanoid:GetState()
            local isRagdolled = humanoid.PlatformStand or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown or state == Enum.HumanoidStateType.Physics
            
            if HitboxEnabled and humanoid.Health > 0 and (not isRagdolled or GroundHitEnabled) then
                hrp.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
                hrp.Transparency = 1 
                hrp.CanCollide = false 
                
                local outline = hrp:FindFirstChild("HitboxOutline")
                if not outline then
                    outline = Instance.new("SelectionBox")
                    outline.Name = "HitboxOutline"
                    outline.Adornee = hrp
                    outline.LineThickness = 0.05
                    outline.Color3 = Color3.new(1, 1, 1) 
                    outline.Parent = hrp
                end
                outline.Transparency = HitboxTrans
            else
                hrp.Size = Vector3.new(2, 2, 1) 
                hrp.Transparency = 1 
                local outline = hrp:FindFirstChild("HitboxOutline")
                if outline then outline:Destroy() end
            end
        end
    end
end)
