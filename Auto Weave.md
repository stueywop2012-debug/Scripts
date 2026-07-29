local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- Prevent duplicate GUIs
if CoreGui:FindFirstChild("XellWareHub") then
    CoreGui.XellWareHub:Destroy()
end

-- ==== GOD WEAVE VARIABLES ====
local AutoWeaveEnabled = false
local WeaveBlinkSpeed = 0.000 -- Absolute max speed
local NextWeaveToggle = 0

-- ==== CUSTOM UI THEME ====
local function applyCustomTheme(element)
    element.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    element.BorderSizePixel = 0
    
    -- Better rounded corners
    local corner = Instance.new("UICorner", element)
    corner.CornerRadius = UDim.new(0, 12)
    
    -- White UI stroke around the edges
    local stroke = Instance.new("UIStroke", element)
    stroke.Color = Color3.fromRGB(255, 255, 255)
    stroke.Thickness = 1.5
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    
    if element:IsA("TextLabel") or element:IsA("TextButton") or element:IsA("TextBox") then
        element.TextColor3 = Color3.fromRGB(255, 255, 255) -- PURE WHITE TEXT
        element.Font = Enum.Font.GothamBold
    end
end

-- ==== UI CONSTRUCTION ====
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XellWareHub"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true 

-- ==== OPEN / CLOSE BUTTON ====
local ToggleUI_Button = Instance.new("TextButton", ScreenGui)
ToggleUI_Button.Size = UDim2.new(0, 110, 0, 35)
ToggleUI_Button.Position = UDim2.new(0, 20, 0, 20) -- Top left corner
ToggleUI_Button.Text = "Close Hub"
ToggleUI_Button.TextSize = 14
applyCustomTheme(ToggleUI_Button)

-- ==== MAIN HUB GUI ====
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -125)
MainFrame.Size = UDim2.new(0, 300, 0, 180)
MainFrame.Active = true
MainFrame.Visible = true 
applyCustomTheme(MainFrame)

-- Toggle UI Logic
ToggleUI_Button.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
    ToggleUI_Button.Text = MainFrame.Visible and "Close Hub" or "Open Hub"
end)

-- Custom Mobile/PC Dragging Logic
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)
MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 35)
applyCustomTheme(TopBar)
TopBar.BackgroundColor3 = Color3.fromRGB(30, 30, 35)

local Title = Instance.new("TextLabel", TopBar)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 15, 0, 0)
Title.Size = UDim2.new(1, -30, 1, 0)
Title.Text = "XellWare // Untouchable Weave"
Title.TextSize = 16
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left

local Toggle_Weave = Instance.new("TextButton", MainFrame)
Toggle_Weave.Size = UDim2.new(1, -20, 0, 40)
Toggle_Weave.Position = UDim2.new(0, 10, 0, 50)
Toggle_Weave.Text = "Auto Weave: OFF"
Toggle_Weave.TextSize = 15
applyCustomTheme(Toggle_Weave)
Toggle_Weave.BackgroundColor3 = Color3.fromRGB(40, 40, 45)

Toggle_Weave.MouseButton1Click:Connect(function()
    AutoWeaveEnabled = not AutoWeaveEnabled
    Toggle_Weave.Text = "Auto Weave: " .. (AutoWeaveEnabled and "ON" or "OFF")
    
    -- Ensure weave is turned off cleanly if disabled
    if not AutoWeaveEnabled and LocalPlayer.Character then
        local core = LocalPlayer.Character:FindFirstChild("Core")
        if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
            core.Communicate[""]:FireServer("Weave", nil, false, nil)
        end
    end
end)

local function createSlider(parent, text, yPos, minVal, maxVal, defaultVal, callback)
    local container = Instance.new("Frame", parent)
    container.Size = UDim2.new(1, -20, 0, 50)
    container.Position = UDim2.new(0, 10, 0, yPos)
    applyCustomTheme(container)
    container.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    
    local label = Instance.new("TextLabel", container)
    label.Size = UDim2.new(1, -10, 0, 20)
    label.Position = UDim2.new(0, 10, 0, 5)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. tostring(defaultVal)
    label.TextSize = 14
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.Font = Enum.Font.GothamBold
    label.TextXAlignment = Enum.TextXAlignment.Left
    
    local track = Instance.new("Frame", container)
    track.Size = UDim2.new(1, -20, 0, 8)
    track.Position = UDim2.new(0, 10, 0, 32)
    track.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    track.BorderSizePixel = 0
    Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)
    
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    fill.BorderSizePixel = 0
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)
    
    local button = Instance.new("TextButton", track)
    button.Size = UDim2.new(1, 0, 1, 14)
    button.Position = UDim2.new(0, 0, 0, -7)
    button.BackgroundTransparency = 1
    button.Text = ""
    
    local drag = false
    local function update(input)
        local pos = math.clamp((input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        local value = minVal + ((maxVal - minVal) * pos)
        value = math.floor(value * 1000) / 1000 
        label.Text = text .. ": " .. tostring(value)
        callback(value)
    end
    
    button.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            drag = true
            update(input)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then drag = false end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if drag and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then update(input) end
    end)
end

-- Max value 0.15, default is 0.000 for pure invincibility
createSlider(MainFrame, "Weave Speed (Untouchable)", 100, 0.000, 0.15, 0.000, function(val) WeaveBlinkSpeed = val end)

-- ==== THE LOCKED-IN GOD WEAVE LOOP ====
RunService.Heartbeat:Connect(function()
    if not LocalPlayer.Character then return end
    
    if AutoWeaveEnabled then
        local now = os.clock()
        if now >= NextWeaveToggle then
            local core = LocalPlayer.Character:FindFirstChild("Core")
            if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                local remote = core.Communicate[""]
                
                -- ZERO GAP LOGIC: 
                -- We fire the 'false' reset, but instead of yielding a frame, we use task.defer.
                -- This forces the 'true' signal into the exact same server tick but perfectly sequenced.
                remote:FireServer("Weave", nil, false, nil)
                
                task.defer(function()
                    remote:FireServer("Weave", nil, true)
                end)
                
                NextWeaveToggle = now + WeaveBlinkSpeed 
            end
        end
    end
end)
