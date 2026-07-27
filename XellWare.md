local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- ==== ENSURE HIT EVENT EXISTS ====
local HitEvent = ReplicatedStorage:FindFirstChild("HitEvent")
if not HitEvent then
    HitEvent = Instance.new("RemoteEvent")
    HitEvent.Name = "HitEvent"
    HitEvent.Parent = ReplicatedStorage
end

-- Prevent duplicate GUIs
if CoreGui:FindFirstChild("XellwareHub") then
    CoreGui.XellwareHub:Destroy()
end

-- ==== VARIABLES ====
local RegularHitboxEnabled = false
local SmartHitboxEnabled = false
local AntiStandEnabled = true 
local HitboxSize = 8.5 
local HitboxTrans = 0.0 
local GroundHitEnabled = false

local AutoWeaveEnabled = false
local CurrentWeaveState = false
local WeaveBlinkSpeed = 0.02
local NextWeaveToggle = 0

local AttackPauseUntil = 0
local HitRegisterPauseTime = 0.50

local isAttacking = false
local hitCharacters = {}
local guiVisible = true

-- ==== ENHANCED UI THEME HELPER ====
local function applyRetroTheme(element, addCorner)
    element.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    
    local stroke = Instance.new("UIStroke", element)
    stroke.Color = Color3.fromRGB(255, 255, 255)
    stroke.Thickness = 1
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    
    if addCorner then
        local corner = Instance.new("UICorner", element)
        corner.CornerRadius = UDim.new(0, 6)
    end
    
    if element:IsA("TextLabel") or element:IsA("TextButton") or element:IsA("TextBox") then
        element.TextColor3 = Color3.new(1, 1, 1)
        element.Font = Enum.Font.Code 
    end
end

-- ==== UI CONSTRUCTION ====
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XellwareHub"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local OpenButton = Instance.new("TextButton", ScreenGui)
OpenButton.Position = UDim2.new(0, 15, 0, 15)
OpenButton.Size = UDim2.new(0, 70, 0, 45)
OpenButton.Text = "Open"
OpenButton.TextSize = 16
OpenButton.Visible = false
applyRetroTheme(OpenButton, true)

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Position = UDim2.new(0.5, -175, 0.5, -160)
MainFrame.Size = UDim2.new(0, 350, 0, 320)
MainFrame.Active = true
MainFrame.Draggable = true 
applyRetroTheme(MainFrame, true)

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 35)
TopBar.BackgroundTransparency = 1

local Title = Instance.new("TextLabel", TopBar)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 15, 0, 0)
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Text = "Xellware Hub - God Mode"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.Code
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseButton = Instance.new("TextButton", TopBar)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Text = "X"
applyRetroTheme(CloseButton, true)

local Line = Instance.new("Frame", MainFrame)
Line.Size = UDim2.new(1, 0, 0, 1)
Line.Position = UDim2.new(0, 0, 0, 35)
Line.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
Line.BorderSizePixel = 0

local TabBar = Instance.new("Frame", MainFrame)
TabBar.Position = UDim2.new(0, 0, 0, 36)
TabBar.Size = UDim2.new(1, 0, 0, 35)
TabBar.BackgroundTransparency = 1

local function createTabButton(text, posOffset, widthPercent)
    local btn = Instance.new("TextButton", TabBar)
    btn.Position = UDim2.new(posOffset, 5, 0, 5)
    btn.Size = UDim2.new(widthPercent, -10, 1, -10) 
    btn.Text = text
    btn.TextSize = 13
    applyRetroTheme(btn, true)
    return btn
end

local TabBtn_Hitbox = createTabButton("Hitbox", 0, 0.33)
local TabBtn_Weave = createTabButton("Weave", 0.33, 0.33)
local TabBtn_Settings = createTabButton("Settings", 0.66, 0.33)

local PagesContainer = Instance.new("Frame", MainFrame)
PagesContainer.BackgroundTransparency = 1
PagesContainer.Position = UDim2.new(0, 10, 0, 80)
PagesContainer.Size = UDim2.new(1, -20, 1, -90)

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
local Page_Settings = createPage("SettingsPage")
Page_Hitbox.Visible = true 

local function createToggle(parent, text, defaultState)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, -10, 0, 35)
    btn.Text = text .. (defaultState and ": ON" or ": OFF")
    btn.TextSize = 15
    applyRetroTheme(btn, true)
    return btn
end

local function createSlider(parent, text, minVal, maxVal, defaultVal, callback)
    local container = Instance.new("Frame", parent)
    container.Size = UDim2.new(1, -10, 0, 45)
    applyRetroTheme(container, true)
    
    local label = Instance.new("TextLabel", container)
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Position = UDim2.new(0, 8, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. tostring(defaultVal)
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.Code
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local track = Instance.new("Frame", container)
    track.Size = UDim2.new(1, -20, 0, 8)
    track.Position = UDim2.new(0, 10, 0, 26)
    track.BackgroundColor3 = Color3.new(0, 0, 0)
    local stroke = Instance.new("UIStroke", track)
    stroke.Color = Color3.new(1, 1, 1)
    stroke.Thickness = 1
    local corner1 = Instance.new("UICorner", track)
    corner1.CornerRadius = UDim.new(1, 0)
    
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.new(1, 1, 1)
    fill.BorderSizePixel = 0
    local corner2 = Instance.new("UICorner", fill)
    corner2.CornerRadius = UDim.new(1, 0)
    
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

local function createLabel(parent, text)
    local lbl = Instance.new("TextLabel", parent)
    lbl.Size = UDim2.new(1, -10, 0, 20)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.new(0.7, 0.7, 0.7)
    lbl.Font = Enum.Font.Code
    lbl.TextSize = 13
    lbl.TextXAlignment = Enum.TextXAlignment.Center
    return lbl
end

-- ==== POPULATING TABS ====
createLabel(Page_Hitbox, "[ Recommended Hitbox Size: 8.5 ]")
local Toggle_RegularHitbox = createToggle(Page_Hitbox, "Regular Hitbox", false)
local Toggle_SmartHitbox = createToggle(Page_Hitbox, "Smart Hitbox (99% Hit Rate)", false)
createSlider(Page_Hitbox, "Hitbox Size", 1.0, 25.0, 8.5, function(val) HitboxSize = val end)
createSlider(Page_Hitbox, "Hide Visuals", 0.0, 1.0, 0.0, function(val) HitboxTrans = val end)
local Toggle_Ground = createToggle(Page_Hitbox, "Hit Grounded (Ragdoll)", false)

local Toggle_Weave = createToggle(Page_Weave, "God Auto Weave", false)
createSlider(Page_Weave, "Blink Speed", 0.005, 0.30, 0.02, function(val) WeaveBlinkSpeed = val end)
createSlider(Page_Weave, "Hit Pause Time", 0.01, 1.00, 0.50, function(val) HitRegisterPauseTime = val end) 

createLabel(Page_Settings, "[ Press SHIFT to hide entire GUI ]")
local Toggle_AntiStand = createToggle(Page_Settings, "Anti-Stand (Fix Head Collisions)", true)

-- ==== UI LOGIC ====
local function switchTab(activePage)
    Page_Hitbox.Visible = false
    Page_Weave.Visible = false
    Page_Settings.Visible = false
    activePage.Visible = true
end

TabBtn_Hitbox.MouseButton1Click:Connect(function() switchTab(Page_Hitbox) end)
TabBtn_Weave.MouseButton1Click:Connect(function() switchTab(Page_Weave) end)
TabBtn_Settings.MouseButton1Click:Connect(function() switchTab(Page_Settings) end)

CloseButton.MouseButton1Click:Connect(function() MainFrame.Visible = false; OpenButton.Visible = true end)
OpenButton.MouseButton1Click:Connect(function() MainFrame.Visible = true; OpenButton.Visible = false end)

Toggle_RegularHitbox.MouseButton1Click:Connect(function() 
    RegularHitboxEnabled = not RegularHitboxEnabled
    Toggle_RegularHitbox.Text = "Regular Hitbox: " .. (RegularHitboxEnabled and "ON" or "OFF")
end)

Toggle_SmartHitbox.MouseButton1Click:Connect(function() 
    SmartHitboxEnabled = not SmartHitboxEnabled
    Toggle_SmartHitbox.Text = "Smart Hitbox (99% Hit Rate): " .. (SmartHitboxEnabled and "ON" or "OFF")
end)

Toggle_AntiStand.MouseButton1Click:Connect(function() 
    AntiStandEnabled = not AntiStandEnabled
    Toggle_AntiStand.Text = "Anti-Stand (Fix Head Collisions): " .. (AntiStandEnabled and "ON" or "OFF")
end)

Toggle_Ground.MouseButton1Click:Connect(function() 
    GroundHitEnabled = not GroundHitEnabled
    Toggle_Ground.Text = "Hit Grounded (Ragdoll): " .. (GroundHitEnabled and "ON" or "OFF")
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

-- ==== INPUT CAPTURE & REVERTED SMART HITBOX ====
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and (input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift) then
        guiVisible = not guiVisible
        ScreenGui.Enabled = guiVisible
    end

    if not gameProcessed then
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            local now = os.clock()
            
            if now >= AttackPauseUntil then
                AttackPauseUntil = now + HitRegisterPauseTime
                isAttacking = true
                
                -- INSTANT WEAVE DROP (Must be unblinked to register damage)
                if AutoWeaveEnabled and CurrentWeaveState and LocalPlayer.Character then
                    local core = LocalPlayer.Character:FindFirstChild("Core")
                    if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                        core.Communicate[""]:FireServer("Weave", nil, false, nil)
                        CurrentWeaveState = false
                    end
                end
                
                if SmartHitboxEnabled then
                    table.clear(hitCharacters)
                    
                    local startTime = os.clock()
                    local swingDuration = math.min(HitRegisterPauseTime, 0.4) 
                    local lastHrpCFrame = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.CFrame
                    
                    local connection
                    connection = RunService.Heartbeat:Connect(function()
                        if os.clock() - startTime >= swingDuration then
                            isAttacking = false
                            if connection then connection:Disconnect() end
                            return
                        end
                        
                        local character = LocalPlayer.Character
                        if not character or not character:FindFirstChild("HumanoidRootPart") then return end
                        
                        local hrp = character.HumanoidRootPart
                        local currentCFrame = hrp.CFrame
                        
                        -- Spatial Hitbox Frame Interpolation (Zero-Delay Version)
                        local distanceMoved = 0
                        local boxCFrame = currentCFrame
                        
                        if lastHrpCFrame then
                            distanceMoved = (currentCFrame.Position - lastHrpCFrame.Position).Magnitude
                            if distanceMoved > 0.01 then
                                boxCFrame = CFrame.lookAt(lastHrpCFrame.Position, currentCFrame.Position) * CFrame.new(0, 0, -distanceMoved / 2)
                            end
                        end
                        
                        local boxSize = Vector3.new(HitboxSize, HitboxSize, HitboxSize + distanceMoved)
                        lastHrpCFrame = currentCFrame 
                        
                        local overlapParams = OverlapParams.new()
                        overlapParams.FilterDescendantsInstances = {character}
                        overlapParams.FilterType = Enum.RaycastFilterType.Exclude
                        
                        local partsInBox = workspace:GetPartBoundsInBox(boxCFrame, boxSize, overlapParams)
                        
                        for _, part in ipairs(partsInBox) do
                            local model = part:FindFirstAncestorOfClass("Model")
                            if model and model ~= character and model:FindFirstChild("Humanoid") then
                                local humanoid = model.Humanoid
                                local state = humanoid:GetState()
                                local isRagdolled = humanoid.PlatformStand or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown or state == Enum.HumanoidStateType.Physics
                                
                                if humanoid.Health > 0 and not hitCharacters[model] then
                                    if not isRagdolled or GroundHitEnabled then
                                        hitCharacters[model] = true 
                                        HitEvent:FireServer(model)
                                    end
                                end
                            end
                        end
                    end)
                else
                    task.delay(0.35, function()
                        isAttacking = false 
                    end)
                end
            end
        end
    end
end)

-- ==== GOD WEAVE BLINK LOOP ====
RunService.Heartbeat:Connect(function()
    if AutoWeaveEnabled and not isAttacking and LocalPlayer.Character then
        local now = os.clock()
        
        if now >= NextWeaveToggle then
            local core = LocalPlayer.Character:FindFirstChild("Core")
            if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                local remote = core.Communicate[""]
                
                if CurrentWeaveState then
                    remote:FireServer("Weave", nil, false, nil)
                    CurrentWeaveState = false
                    NextWeaveToggle = now + WeaveBlinkSpeed 
                else
                    remote:FireServer("Weave", nil, true)
                    CurrentWeaveState = true
                    NextWeaveToggle = now + WeaveBlinkSpeed 
                end
            end
        end
    end
end)

-- ==== ANTI-STAND COLLISION FIX ====
RunService.Stepped:Connect(function()
    if RegularHitboxEnabled and AntiStandEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                local head = player.Character:FindFirstChild("Head")
                
                if hrp then hrp.CanCollide = false end
                if head then head.CanCollide = false end
            end
        end
    end
end)

-- ==== REGULAR HITBOX & VISUAL ESP LOOP ====
RunService.RenderStepped:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            
            local hrp = player.Character.HumanoidRootPart
            local humanoid = player.Character.Humanoid
            
            if RegularHitboxEnabled and humanoid.Health > 0 then
                hrp.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
                hrp.Transparency = 1 
                
                local outline = hrp:FindFirstChild("HitboxOutline")
                if not outline then
                    outline = Instance.new("SelectionBox")
                    outline.Name = "HitboxOutline"
                    outline.Adornee = hrp
                    outline.LineThickness = 0.05 
                    outline.Color3 = Color3.new(1, 0, 0)
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
