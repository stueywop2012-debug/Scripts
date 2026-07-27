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
local AutoWeaveEnabled = false
local CurrentWeaveState = false
local WeaveBlinkSpeed = 0.02 
local NextWeaveToggle = 0

local AttackPauseUntil = 0
local HitRegisterPauseTime = 0.50

-- ==== MODERN CUSTOM UI THEME ====
local function applyModernTheme(element)
    element.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    element.BorderSizePixel = 0
    
    local corner = Instance.new("UICorner", element)
    corner.CornerRadius = UDim.new(0, 6)
    
    if element:IsA("TextLabel") or element:IsA("TextButton") or element:IsA("TextBox") then
        element.TextColor3 = Color3.fromRGB(240, 240, 240)
        element.Font = Enum.Font.GothamMedium 
    end
end

-- ==== UI CONSTRUCTION ====
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XellwareHub"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local OpenButton = Instance.new("TextButton", ScreenGui)
OpenButton.Position = UDim2.new(0, 10, 0, 10)
OpenButton.Size = UDim2.new(0, 60, 0, 35)
OpenButton.Text = "Open"
OpenButton.TextSize = 14
OpenButton.Visible = false
applyModernTheme(OpenButton)

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -180)
MainFrame.Size = UDim2.new(0, 300, 0, 360) 
MainFrame.Active = true
MainFrame.Draggable = true 
applyModernTheme(MainFrame)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 30)
applyModernTheme(TopBar)
TopBar.BackgroundColor3 = Color3.fromRGB(40, 40, 45)

local Title = Instance.new("TextLabel", TopBar)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 10, 0, 0)
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Text = "XellWare - God Mode"
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseButton = Instance.new("TextButton", TopBar)
CloseButton.Position = UDim2.new(1, -30, 0, 3)
CloseButton.Size = UDim2.new(0, 24, 0, 24)
CloseButton.Text = "X"
CloseButton.TextSize = 12
applyModernTheme(CloseButton)
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)

-- Pages Container (Single Page layout since Hitbox tab is removed)
local PagesContainer = Instance.new("Frame", MainFrame)
PagesContainer.BackgroundTransparency = 1
PagesContainer.Position = UDim2.new(0, 10, 0, 45)
PagesContainer.Size = UDim2.new(1, -20, 1, -55)

local function createPage(name)
    local page = Instance.new("ScrollingFrame", PagesContainer)
    page.Name = name
    page.BackgroundTransparency = 1
    page.Size = UDim2.new(1, 0, 1, 0)
    page.ScrollBarThickness = 3
    page.Visible = true
    local layout = Instance.new("UIListLayout", page)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 6)
    return page
end

local Page_Weave = createPage("WeavePage")

-- Generators
local function createToggle(parent, text, defaultState)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Text = text .. (defaultState and ": ON" or ": OFF")
    btn.TextSize = 13
    applyModernTheme(btn)
    return btn
end

local function createSlider(parent, text, minVal, maxVal, defaultVal, callback)
    local container = Instance.new("Frame", parent)
    container.Size = UDim2.new(1, -10, 0, 40)
    applyModernTheme(container)
    
    local label = Instance.new("TextLabel", container)
    label.Size = UDim2.new(1, -10, 0, 18)
    label.Position = UDim2.new(0, 8, 0, 2)
    label.BackgroundTransparency = 1
    
    local function getSpeedText(val)
        if text ~= "Blink Speed" then return "" end
        if val <= 0.05 then return " [FAST]"
        elseif val <= 0.15 then return " [MEDIUM]"
        else return " [SLOW]" end
    end

    label.Text = text .. getSpeedText(defaultVal) .. ": " .. tostring(defaultVal)
    label.TextSize = 12
    label.Font = Enum.Font.GothamMedium
    label.TextColor3 = Color3.fromRGB(240, 240, 240)
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local track = Instance.new("Frame", container)
    track.Size = UDim2.new(1, -20, 0, 6)
    track.Position = UDim2.new(0, 10, 0, 25)
    track.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    local corner = Instance.new("UICorner", track)
    corner.CornerRadius = UDim.new(1, 0)
    
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(150, 150, 255)
    local fillCorner = Instance.new("UICorner", fill)
    fillCorner.CornerRadius = UDim.new(1, 0)
    
    local button = Instance.new("TextButton", track)
    button.Size = UDim2.new(1, 0, 1, 10)
    button.Position = UDim2.new(0, 0, 0, -5)
    button.BackgroundTransparency = 1
    button.Text = ""
    
    local dragging = false
    
    local function update(input)
        local pos = math.clamp((input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        local value = minVal + ((maxVal - minVal) * pos)
        value = math.floor(value * 100) / 100 
        label.Text = text .. getSpeedText(value) .. ": " .. tostring(value)
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

-- Populate God Weave Elements
local Toggle_Weave = createToggle(Page_Weave, "God Auto Weave", false)
createSlider(Page_Weave, "Blink Speed", 0.005, 0.30, 0.02, function(val) WeaveBlinkSpeed = val end)
createSlider(Page_Weave, "Hit Pause Time", 0.01, 1.00, 0.50, function(val) HitRegisterPauseTime = val end) 

-- ==== UI LOGIC ====
CloseButton.MouseButton1Click:Connect(function() MainFrame.Visible = false; OpenButton.Visible = true end)
OpenButton.MouseButton1Click:Connect(function() MainFrame.Visible = true; OpenButton.Visible = false end)

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
            local now = os.clock()
            
            if now >= AttackPauseUntil then
                AttackPauseUntil = now + HitRegisterPauseTime
                
                if AutoWeaveEnabled and CurrentWeaveState and LocalPlayer.Character then
                    local core = LocalPlayer.Character:FindFirstChild("Core")
                    if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                        core.Communicate[""]:FireServer("Weave", nil, false, nil)
                        CurrentWeaveState = false
                    end
                end
            end
        end
    end
end)

-- ==== GOD WEAVE BLINK LOOP ====
RunService.Heartbeat:Connect(function()
    if AutoWeaveEnabled and LocalPlayer.Character then
        local now = os.clock()
        
        if now >= NextWeaveToggle then
            local core = LocalPlayer.Character:FindFirstChild("Core")
            if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                local remote = core.Communicate[""]
                
                if CurrentWeaveState then
                    remote:FireServer("Weave", nil, false, nil)
                    CurrentWeaveState = false
                    NextWeaveToggle = now 
                else
                    remote:FireServer("Weave", nil, true)
                    CurrentWeaveState = true
                    NextWeaveToggle = now + WeaveBlinkSpeed
                end
            end
        end
    end
end)
