Endless GAMES [NO ADS] - Universal AutoFarm & Leaderboard Submitter (Fixed & Optimized) ]] local Players = game:GetService("Players") local UserInputService = game:GetService("UserInputService") local LocalPlayer = Players.LocalPlayer local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
for
_, c in ipairs(PlayerGui:GetChildren())
do
  
  if
  c.Name == "EndlessGamesAutofarm"
  then
    c:Destroy()
  end
  
end
local gscProto = require(LocalPlayer.PlayerScripts.TS.controllers["game-session-controller"]).GameSessionController local gsc = nil
for
_, obj in ipairs(getgc(true))
do
  
  if
  type(obj) == "table" and getmetatable(obj) == gscProto
  then
    gsc = obj break
  end
  
end

if
not gsc
then
  warn("[AutoFarm] GameSessionController not found!") return
end
local ScreenGui = Instance.new("ScreenGui") ScreenGui.Name = "EndlessGamesAutofarm" ScreenGui.ResetOnSpawn = false ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling ScreenGui.Parent = PlayerGui local MainFrame = Instance.new("Frame") MainFrame.Name = "MainFrame" MainFrame.Size = UDim2.new(0, 310, 0, 380) MainFrame.Position = UDim2.new(0.03, 0, 0.25, 0) MainFrame.BackgroundColor3 = Color3.fromRGB(22, 24, 30) MainFrame.BorderSizePixel = 0 MainFrame.Active = true MainFrame.Parent = ScreenGui local MainCorner = Instance.new("UICorner") MainCorner.CornerRadius = UDim.new(0, 10) MainCorner.Parent = MainFrame local MainStroke = Instance.new("UIStroke") MainStroke.Color = Color3.fromRGB(55, 62, 78) MainStroke.Thickness = 1.5 MainStroke.Parent = MainFrame local dragging, dragInput, dragStart, startPos MainFrame.InputBegan:Connect(
function
  (input)
  if
  input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch
  then
    dragging = true dragStart = input.Position startPos = MainFrame.Position input.Changed:Connect(
    function
      ()
      if
      input.UserInputState == Enum.UserInputState.End
      then
        dragging = false
      end
      
    end
    )
  end
  
end
) MainFrame.InputChanged:Connect(
function
  (input)
  if
  input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch
  then
    dragInput = input
  end
  
end
) UserInputService.InputChanged:Connect(
function
  (input)
  if
  input == dragInput and dragging
  then
    local delta = input.Position - dragStart MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
  end
  
end
) local Header = Instance.new("Frame") Header.Size = UDim2.new(1, 0, 0, 38) Header.BackgroundColor3 = Color3.fromRGB(16, 18, 24) Header.BorderSizePixel = 0 Header.Parent = MainFrame local HeaderCorner = Instance.new("UICorner") HeaderCorner.CornerRadius = UDim.new(0, 10) HeaderCorner.Parent = Header local Title = Instance.new("TextLabel") Title.Size = UDim2.new(1, -16, 1, 0) Title.Position = UDim2.new(0, 12, 0, 0) Title.BackgroundTransparency = 1 Title.Text = "Endless GAMES AutoFarm" Title.Font = Enum.Font.GothamBold Title.TextColor3 = Color3.fromRGB(245, 245, 255) Title.TextSize = 14 Title.TextXAlignment = Enum.TextXAlignment.Left Title.Parent = Header local Content = Instance.new("Frame") Content.Size = UDim2.new(1, -20, 1, -48) Content.Position = UDim2.new(0, 10, 0, 44) Content.BackgroundTransparency = 1 Content.Parent = MainFrame local UIList = Instance.new("UIListLayout") UIList.Padding = UDim.new(0, 7) UIList.SortOrder = Enum.SortOrder.LayoutOrder UIList.Parent = Content local StatusBox = Instance.new("Frame") StatusBox.Size = UDim2.new(1, 0, 0, 48) StatusBox.BackgroundColor3 = Color3.fromRGB(30, 34, 44) StatusBox.BorderSizePixel = 0 StatusBox.Parent = Content local StatusCorner = Instance.new("UICorner") StatusCorner.CornerRadius = UDim.new(0, 6) StatusCorner.Parent = StatusBox local StatusLabel = Instance.new("TextLabel") StatusLabel.Size = UDim2.new(1, -16, 0, 20) StatusLabel.Position = UDim2.new(0, 8, 0, 3) StatusLabel.BackgroundTransparency = 1 StatusLabel.Text = "Status: Ready (Idle in Hub)" StatusLabel.Font = Enum.Font.GothamSemibold StatusLabel.TextColor3 = Color3.fromRGB(180, 190, 210) StatusLabel.TextSize = 12 StatusLabel.TextXAlignment = Enum.TextXAlignment.Left StatusLabel.Parent = StatusBox local ScoreLabel = Instance.new("TextLabel") ScoreLabel.Size = UDim2.new(1, -16, 0, 18) ScoreLabel.Position = UDim2.new(0, 8, 0, 24) ScoreLabel.BackgroundTransparency = 1 ScoreLabel.Text = "Score: 0" ScoreLabel.Font = Enum.Font.GothamBold ScoreLabel.TextColor3 = Color3.fromRGB(120, 225, 130) ScoreLabel.TextSize = 12 ScoreLabel.TextXAlignment = Enum.TextXAlignment.Left ScoreLabel.Parent = StatusBox local SelectLabel = Instance.new("TextLabel") SelectLabel.Size = UDim2.new(1, 0, 0, 16) SelectLabel.BackgroundTransparency = 1 SelectLabel.Text = "Select Mini-Game:" SelectLabel.Font = Enum.Font.GothamBold SelectLabel.TextColor3 = Color3.fromRGB(205, 210, 225) SelectLabel.TextSize = 12 SelectLabel.TextXAlignment = Enum.TextXAlignment.Left SelectLabel.Parent = Content local supportedGames = { { id = "knife-hit", name = "Knife Combo / Knife Hit (Best)" }, { id = "timberman", name = "Chop Chop / Timberman (Fast)" }, { id = "stack", name = "Stack It / Stack (Blocks)" } } local selectedGameId = "knife-hit" local gameButtons = {}
for
_, g in ipairs(supportedGames)
do
  local btn = Instance.new("TextButton") btn.Size = UDim2.new(1, 0, 0, 26) btn.BackgroundColor3 = (g.id == selectedGameId) and Color3.fromRGB(45, 105, 210) or Color3.fromRGB(34, 38, 50) btn.Text = " " .. g.name btn.Font = Enum.Font.GothamMedium btn.TextColor3 = Color3.fromRGB(240, 240, 255) btn.TextSize = 11 btn.TextXAlignment = Enum.TextXAlignment.Left btn.AutoButtonColor = false btn.BorderSizePixel = 0 btn.Parent = Content local bCorner = Instance.new("UICorner") bCorner.CornerRadius = UDim.new(0, 5) bCorner.Parent = btn btn.MouseButton1Click:Connect(
  function
    () selectedGameId = g.id
    for
    id, b in pairs(gameButtons)
    do
      b.BackgroundColor3 = (id == selectedGameId) and Color3.fromRGB(45, 105, 210) or Color3.fromRGB(34, 38, 50)
    end
    
  end
  ) gameButtons[g.id] = btn
end
local ScoreGoalLabel = Instance.new("TextLabel") ScoreGoalLabel.Size = UDim2.new(1, 0, 0, 16) ScoreGoalLabel.BackgroundTransparency = 1 ScoreGoalLabel.Text = "Target Score (Leaderboard Goal):" ScoreGoalLabel.Font = Enum.Font.GothamBold ScoreGoalLabel.TextColor3 = Color3.fromRGB(205, 210, 225) ScoreGoalLabel.TextSize = 12 ScoreGoalLabel.TextXAlignment = Enum.TextXAlignment.Left ScoreGoalLabel.Parent = Content local InputBox = Instance.new("TextBox") InputBox.Size = UDim2.new(1, 0, 0, 28) InputBox.BackgroundColor3 = Color3.fromRGB(34, 38, 50) InputBox.BorderSizePixel = 0 InputBox.Text = "120" InputBox.Font = Enum.Font.GothamBold InputBox.TextColor3 = Color3.fromRGB(255, 215, 0) InputBox.TextSize = 13 InputBox.ClearTextOnFocus = false InputBox.Parent = Content local InputCorner = Instance.new("UICorner") InputCorner.CornerRadius = UDim.new(0, 5) InputCorner.Parent = InputBox local StartButton = Instance.new("TextButton") StartButton.Size = UDim2.new(1, 0, 0, 36) StartButton.BackgroundColor3 = Color3.fromRGB(35, 155, 80) StartButton.BorderSizePixel = 0 StartButton.Text = "START AUTOFARM" StartButton.Font = Enum.Font.GothamBold StartButton.TextColor3 = Color3.fromRGB(255, 255, 255) StartButton.TextSize = 13 StartButton.Parent = Content local StartCorner = Instance.new("UICorner") StartCorner.CornerRadius = UDim.new(0, 7) StartCorner.Parent = StartButton local ExitHubButton = Instance.new("TextButton") ExitHubButton.Size = UDim2.new(1, 0, 0, 26) ExitHubButton.BackgroundColor3 = Color3.fromRGB(50, 55, 70) ExitHubButton.BorderSizePixel = 0 ExitHubButton.Text = "Return To Hub" ExitHubButton.Font = Enum.Font.GothamMedium ExitHubButton.TextColor3 = Color3.fromRGB(215, 215, 230) ExitHubButton.TextSize = 11 ExitHubButton.Parent = Content local ExitCorner = Instance.new("UICorner") ExitCorner.CornerRadius = UDim.new(0, 5) ExitCorner.Parent = ExitHubButton local farming = false ExitHubButton.MouseButton1Click:Connect(
function
  () farming = false StartButton.Text = "START AUTOFARM" StartButton.BackgroundColor3 = Color3.fromRGB(35, 155, 80) StatusLabel.Text = "Status: Exiting to Hub..." pcall(
  function
    () gsc:exitToHub()
  end
  ) StatusLabel.Text = "Status: In Hub (Ready)"
end
) StartButton.MouseButton1Click:Connect(
function
  ()
  if
  farming
  then
    farming = false StartButton.Text = "START AUTOFARM" StartButton.BackgroundColor3 = Color3.fromRGB(35, 155, 80) StatusLabel.Text = "Status: Stopped by user" return
  end
  local targetScore = tonumber(InputBox.Text) or 100 farming = true StartButton.Text = "STOP AUTOFARM" StartButton.BackgroundColor3 = Color3.fromRGB(195, 45, 45) task.spawn(
  function
    () StatusLabel.Text = "Status: Launching " .. selectedGameId .. "..." pcall(
    function
      () gsc:exitToHub()
    end
    ) task.wait(0.6) gsc:launch({ gameId = selectedGameId, mode = "regular" }) task.wait(1.5) local inst = gsc.instance
    if
    not inst
    then
      StatusLabel.Text = "Status: Failed to load instance" farming = false StartButton.Text = "START AUTOFARM" StartButton.BackgroundColor3 = Color3.fromRGB(35, 155, 80) return
    end
    StatusLabel.Text = "Status: Farming to " .. tostring(targetScore) .. "..."
    if
    selectedGameId == "knife-hit"
    then
      
      while
      farming and inst and not inst.dead and inst.score < targetScore
      do
        inst.stuck = {} inst:onPress() ScoreLabel.Text = "Score: " .. tostring(inst.score) .. " / " .. tostring(targetScore) task.wait(0.12)
      end
      
      if
      inst and not inst.dead
      then
        StatusLabel.Text = "Status: Target reached! Submitting score..." inst:die()
      end
      
    elseif
      selectedGameId == "timberman"
      then
        
        while
        farming and inst and not inst.dead and inst.score < targetScore
        do
          inst.branchSides[1] = "none" inst.branchSides[2] = "none" inst:resolveChop(inst.playerSide or "left") ScoreLabel.Text = "Score: " .. tostring(inst.score) .. " / " .. tostring(targetScore) task.wait(0.05)
        end
        
        if
        inst and not inst.dead
        then
          StatusLabel.Text = "Status: Target reached! Submitting score..." inst:die("branch")
        end
        
      elseif
        selectedGameId == "stack"
        then
          
          while
          farming and inst and not inst.dead and inst.score < targetScore
          do
            inst.movingOffset = 0 inst:drop() ScoreLabel.Text = "Score: " .. tostring(inst.score) .. " / " .. tostring(targetScore) task.wait(0.06)
          end
          
          if
          inst and not inst.dead
          then
            StatusLabel.Text = "Status: Target reached! Submitting score..." inst:die()
          end
          
        end
        task.wait(1.5) StatusLabel.Text = "Status: Done! Score sent to leaderboard." farming = false StartButton.Text = "START AUTOFARM" StartButton.BackgroundColor3 = Color3.fromRGB(35, 155, 80)
      end
      )
    end
    )
