local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

if CoreGui:FindFirstChild("XellwareWeave") then CoreGui.XellwareWeave:Destroy() end

local AutoWeaveEnabled = false
local CurrentWeaveState = false
local WeaveBlinkSpeed = 0.01 
local NextWeaveToggle = 0
local isAttacking = false
local AttackPauseUntil = 0
local guiVisible = true

local function applyRetroTheme(element, isButton)
    element.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    local stroke = Instance.new("UIStroke", element)
    stroke.Color = Color3.fromRGB(200, 200, 200)
    stroke.Thickness = 2
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    local corner = Instance.new("UICorner", element)
    corner.CornerRadius = UDim.new(0, 0)
    
    if element:IsA("TextLabel") or element:IsA("TextButton") then
        element.TextColor3 = Color3.new(1, 1, 1)
        element.Font = Enum.Font.Code 
    end
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XellwareWeave"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

local OpenButton = Instance.new("TextButton", ScreenGui)
OpenButton.Position = UDim2.new(0, 120, 0, 20) -- Offset to not overlap Hitbox button
OpenButton.Size = UDim2.new(0, 80, 0, 40)
OpenButton.Text = "[ WEAVE ]"
OpenButton.TextSize = 14
OpenButton.Visible = false
applyRetroTheme(OpenButton, true)

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Position = UDim2.new(0.5, 10, 0.5, -180) -- Slightly offset to right
MainFrame.Size = UDim2.new(0, 250, 0, 200)
MainFrame.Active = true
MainFrame.Draggable = true 
applyRetroTheme(MainFrame, false)

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 35)
TopBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TopBar.BorderSizePixel = 0

local Title = Instance.new("TextLabel", TopBar)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 15, 0, 0)
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Text = "XELLWARE - WEAVE"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.Code
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left

local CloseButton = Instance.new("TextButton", TopBar)
CloseButton.Position = UDim2.new(1, -35, 0, 5)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Text = "X"
applyRetroTheme(CloseButton, true)
CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)

local Container = Instance.new("Frame", MainFrame)
Container.BackgroundTransparency = 1
Container.Position = UDim2.new(0, 10, 0, 45)
Container.Size = UDim2.new(1, -20, 1, -55)

CloseButton.MouseButton1Click:Connect(function() MainFrame.Visible = false; OpenButton.Visible = true end)
OpenButton.MouseButton1Click:Connect(function() MainFrame.Visible = true; OpenButton.Visible = false end)

local ToggleWeaveBtn = Instance.new("TextButton", Container)
ToggleWeaveBtn.Size = UDim2.new(1, 0, 0, 38)
ToggleWeaveBtn.Position = UDim2.new(0, 0, 0, 0)
ToggleWeaveBtn.Text = "God Auto Weave [OFF]"
ToggleWeaveBtn.TextSize = 15
applyRetroTheme(ToggleWeaveBtn, true)

ToggleWeaveBtn.MouseButton1Click:Connect(function()
    AutoWeaveEnabled = not AutoWeaveEnabled
    ToggleWeaveBtn.Text = "God Auto Weave" .. (AutoWeaveEnabled and " [ON]" or " [OFF]")
    ToggleWeaveBtn.TextColor3 = AutoWeaveEnabled and Color3.fromRGB(100, 255, 100) or Color3.new(1, 1, 1)
    
    if not AutoWeaveEnabled and LocalPlayer.Character then
        local core = LocalPlayer.Character:FindFirstChild("Core")
        if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
            core.Communicate[""]:FireServer("Weave", nil, false, nil)
            CurrentWeaveState = false
        end
    end
end)

local sliderFrame = Instance.new("Frame", Container)
sliderFrame.Size = UDim2.new(1, 0, 0, 50)
sliderFrame.Position = UDim2.new(0, 0, 0, 50)
applyRetroTheme(sliderFrame, false)

local sliderLabel = Instance.new("TextLabel", sliderFrame)
sliderLabel.Size = UDim2.new(1, -10, 0, 20)
sliderLabel.Position = UDim2.new(0, 8, 0, 2)
sliderLabel.BackgroundTransparency = 1
sliderLabel.Text = "Blink Speed: 0.01"
sliderLabel.TextColor3 = Color3.new(1, 1, 1)
sliderLabel.Font = Enum.Font.Code
sliderLabel.TextSize = 14
sliderLabel.TextXAlignment = Enum.TextXAlignment.Left

local track = Instance.new("Frame", sliderFrame)
track.Size = UDim2.new(1, -20, 0, 12)
track.Position = UDim2.new(0, 10, 0, 28)
track.BackgroundColor3 = Color3.new(0, 0, 0)
local stroke = Instance.new("UIStroke", track)
stroke.Color = Color3.new(1, 1, 1)

local fill = Instance.new("Frame", track)
fill.Size = UDim2.new((0.01 - 0.001) / (0.10 - 0.001), 0, 1, 0)
fill.BackgroundColor3 = Color3.new(1, 1, 1)
fill.BorderSizePixel = 0

local sBtn = Instance.new("TextButton", track)
sBtn.Size = UDim2.new(1, 0, 1, 0)
sBtn.BackgroundTransparency = 1
sBtn.Text = ""

local dragging = false
sBtn.InputBegan:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        local function update(input)
            local pos = math.clamp((input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
            fill.Size = UDim2.new(pos, 0, 1, 0)
            WeaveBlinkSpeed = math.floor((0.001 + ((0.10 - 0.001) * pos)) * 1000) / 1000 
            sliderLabel.Text = "Blink Speed: " .. tostring(WeaveBlinkSpeed)
        end
        update(inp)
    end
end)
UserInputService.InputEnded:Connect(function(inp) if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then dragging = false end end)
UserInputService.InputChanged:Connect(function(inp) if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch) then
    local pos = math.clamp((inp.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
    fill.Size = UDim2.new(pos, 0, 1, 0)
    WeaveBlinkSpeed = math.floor((0.001 + ((0.10 - 0.001) * pos)) * 1000) / 1000 
    sliderLabel.Text = "Blink Speed: " .. tostring(WeaveBlinkSpeed)
end end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and (input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift) then
        guiVisible = not guiVisible
        ScreenGui.Enabled = guiVisible
    end
    if not gameProcessed and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
        local now = os.clock()
        if now >= AttackPauseUntil then
            AttackPauseUntil = now + 0.40
            isAttacking = true
            
            if AutoWeaveEnabled and CurrentWeaveState and LocalPlayer.Character then
                local core = LocalPlayer.Character:FindFirstChild("Core")
                if core and core:FindFirstChild("Communicate") and core.Communicate:FindFirstChild("") then
                    core.Communicate[""]:FireServer("Weave", nil, false, nil)
                    CurrentWeaveState = false
                    task.wait(0.06) 
                end
            end
            
            task.delay(0.35, function()
                isAttacking = false 
            end)
        end
    end
end)

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
