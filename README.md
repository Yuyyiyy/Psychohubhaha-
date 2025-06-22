-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local gui = LocalPlayer:WaitForChild("PlayerGui")

-- CONFIGS
getgenv().configs = getgenv().configs or {
 Size = Vector3.new(1e9000, 1e9000, 1e9000)
}
local Ignorelist = OverlapParams.new()
Ignorelist.FilterType = Enum.RaycastFilterType.Include

-- STABLE WAIT OVERRIDES & HOOKS
local function stableWait()
 RunService.Heartbeat:Wait()
 return nil
end

-- Hook wait, task.wait, delay, spawn — 10 times each
for i = 1, 10 do
 pcall(function()
  hookfunction(wait, stableWait)
  hookfunction(task.wait, stableWait)
  hookfunction(delay, function(_, func) task.spawn(func) return nil end)
  hookfunction(spawn, function(func) task.spawn(func) end)
 end)
end

-- Additional hook on wait/task.wait if hookfunction exists
if hookfunction then
 for i = 1, 10 do
  wait = hookfunction(wait, function() RunService.Heartbeat:Wait() return 0 end)
  task.wait = function() RunService.Heartbeat:Wait() return 0 end
 end
end

-- MASSIVE NO-YIELD HOOK SPAM (57x)
for i = 1, 57 do
 pcall(function()
  hookfunction(function() return i end, function(...) return nil end)
 end)
end

-- KEYWORD MATCH KILL/SPEED OVERRIDES (10 hookfunctions)
local keywords = {
 "kill", "damage", "hit", "attack", "strike", "explode", "instantkill",
 "toolattack", "wait", "delay"
}

for _,v in ipairs(getgc(true)) do
 if typeof(v) == "function" and islclosure(v) then
  local info = debug.getinfo(v)
  if info.name then
   local lowerName = info.name:lower()
   for _, keyword in pairs(keywords) do
    if lowerName:find(keyword) then
     -- Hook each matching function 10 times total (1 hook each, 10 keywords)
     for i = 1, 1 do
      pcall(function() hookfunction(v, function(...) return nil end) end)
     end
     break
    end
   end
  end
 end
end

-- TOOL / CHARACTER SHIELD — hook 10 times each
for i = 1, 10 do
 pcall(function()
  hookfunction(game.Destroy, function(inst, ...)
   if inst:IsA("Humanoid") or inst:IsA("Tool") then return end
   return game.Destroy(inst, ...)
  end)
 end)
end

for i = 1, 10 do
 pcall(function()
  hookfunction(workspace.BreakJoints, function(...) return end)
 end)
end

-- BASIC SHIELDS — hook 10 times each
for i = 1, 10 do
 pcall(function()
  hookfunction(error, function(...) return nil end)
  hookfunction(assert, function(cond, msg) return cond or true end)
  hookfunction(Instance.new, function(className, ...)
   if className == "Explosion" or className == "Smoke" or className == "Sound" then
    return Instance.new("Part")
   end
   return Instance.new(className, ...)
  end)
 end)
end

pcall(function()
 for i = 1, 10 do
  hookfunction(game:GetService("StarterGui").SetCore, function(...) return end)
 end
end)

-- REMOTE BLOCKS / COOLDOWN KILL DENY — hook 10 times each
for _,v in ipairs(getgc(true)) do
 if typeof(v) == "function" and islclosure(v) then
  local info = debug.getinfo(v)
  if info.name and info.name:lower():find("invoke") then
   for i = 1, 10 do
    pcall(function() hookfunction(v, function(...) return nil end) end)
   end
  end
 end
end

-- EXTRA REMOTE KILL PROTECTION — hook 10 times
pcall(function()
 for i = 1, 10 do
  hookfunction(ReplicatedStorage:WaitForChild("KillFunction"), function(...) return nil end)
 end
end)

-- FAST OVERRIDES — hook 10 times
for i = 1, 10 do
 pcall(function()
  hookfunction(function() return true end, function(...) return true end)
  hookfunction(Players:GetChildren(), function(...) return nil end)
  hookfunction(workspace:GetChildren(), function(...) return nil end)
 end)
end

-- METATABLE SHIELDS (kept 1x to avoid instability)
local mt = getrawmetatable(game)
setreadonly(mt, false)
local oldNamecall = mt.__namecall
local oldIndex = mt.__newindex

mt.__namecall = newcclosure(function(self, ...)
 if tostring(self) == "Kick" or getnamecallmethod() == "Kick" then return end
 return oldNamecall(self, ...)
end)

mt.__newindex = newcclosure(function(t, k, v)
 if tostring(t) == "Humanoid" and (k == "WalkSpeed" or k == "JumpPower") then return end
 return oldIndex(t, k, v)
end)
setreadonly(mt, true)

-- ULTRA FORCE SPEED + MOTION
local function forceSpeed()
 local char = LocalPlayer.Character
 if not char then return end
 local humanoid = char:FindFirstChildOfClass("Humanoid")
 if humanoid then
  humanoid.WalkSpeed = 1e6
  humanoid.JumpPower = 1e6
  humanoid.AutoRotate = false
  humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
  humanoid:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, false)
  humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)

  for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
   pcall(function()
    track:AdjustSpeed(9999999)
   end)
  end
 end
end

RunService.Heartbeat:Connect(forceSpeed)

warn("[🌌⚡] GODSPEED OVERDRIVE SCRIPT — 10 Hooks | Motion Max | Instant Kill Enabled | NO WAIT WORLD!")

-------------------------------------------
-- ESP + FOLLOW + HITBOX (optimized)
-------------------------------------------

-- Settings
_G.HeadSize = 15
_G.HitboxSize = Vector3.new(20, 20, 20) -- Bigger and visible hitbox
_G.Disabled = false
_G.ToggleKey = Enum.KeyCode.H
local detectionRange = 25
local frameUpdateRate = 0.1  -- Update every 0.1s to reduce lag

-- GUI Setup
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 50)
frame.Position = UDim2.new(0.5, -100, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
frame.BackgroundTransparency = 0.5
frame.Visible = false
frame.Parent = gui

local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.TextColor3 = Color3.fromRGB(255, 255, 255)
label.TextScaled = true
label.Parent = frame

-- Find closest target within detectionRange
local function findClosestTarget()
 local closestPlayer, minDistance = nil, detectionRange
 if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return nil end

 for _, v in pairs(Players:GetPlayers()) do
  if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
   local dist = (LocalPlayer.Character.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
   if dist <= minDistance then
    minDistance = dist
    closestPlayer = v
   end
  end
 end
 return closestPlayer
end

-- Follow closest target's HumanoidRootPart by moving player's tools there
local following = false
local lastFollowUpdate = 0

local function followTarget()
 if following then return end
 following = true

 RunService.Heartbeat:Connect(function()
  if tick() - lastFollowUpdate < frameUpdateRate or _G.Disabled then return end
  lastFollowUpdate = tick()

  local closestEnemy = findClosestTarget()
  if closestEnemy and closestEnemy.Character and closestEnemy.Character:FindFirstChild("HumanoidRootPart") then
   label.Text = "Following: " .. closestEnemy.Name
   frame.Visible = true

   local targetHRP = closestEnemy.Character.HumanoidRootPart
   for _, tool in pairs(LocalPlayer.Character:GetChildren()) do
    if tool:IsA("Tool") then
     local handle = tool:FindFirstChild("Handle")
     if handle then
      handle.Position = targetHRP.Position + Vector3.new(0, 2, 0)
     elseif tool.PrimaryPart then
      tool:SetPrimaryPartCFrame(targetHRP.CFrame * CFrame.new(0, 2, 0))
     end
    end
   end
  else
   frame.Visible = false
  end
 end)
end

followTarget()

-- ESP + Hitbox + Emoji + Health Text
local lastESPUpdate = 0
RunService.RenderStepped:Connect(function()
 if tick() - lastESPUpdate < frameUpdateRate or _G.Disabled then return end
 lastESPUpdate = tick()

 for _, v in pairs(Players:GetPlayers()) do
  if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Head") then
   pcall(function()
    local head = v.Character.Head
    -- Customize Head
    head.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
    head.Transparency = 1
    head.BrickColor = BrickColor.new("Really black")
    head.Material = Enum.Material.Neon
    head.CanCollide = false
    head.Massless = true

    -- Setup Hitbox
    local hitbox = v.Character:FindFirstChild("Hitbox")
    if not hitbox then
     hitbox = Instance.new("Part")
     hitbox.Name = "Hitbox"
     hitbox.Size = _G.HitboxSize
     hitbox.Transparency = 0.3
     hitbox.BrickColor = BrickColor.new("Bright red")
     hitbox.Material = Enum.Material.Neon
     hitbox.Anchored = true
     hitbox.CanCollide = false
     hitbox.Massless = true
     hitbox.Parent = v.Character

     -- Emoji GUI
     local emoji = Instance.new("BillboardGui")
     emoji.Name = "Emoji"
     emoji.Size = UDim2.new(10, 0, 10, 0)
     emoji.Adornee = hitbox
     emoji.AlwaysOnTop = true
     emoji.Parent = hitbox

     local emojiText = Instance.new("TextLabel")
     emojiText.Name = "EmojiLabel"
     emojiText.Size = UDim2.new(1, 0, 1, 0)
     emojiText.BackgroundTransparency = 1
     emojiText.TextScaled = true
     emojiText.Font = Enum.Font.GothamBlack
     emojiText.TextColor3 = Color3.fromRGB(255, 255, 0)
     emojiText.TextStrokeTransparency = 0
     emojiText.Text = "😂"
     emojiText.Parent = emoji

     -- Health GUI
     local healthGui = Instance.new("BillboardGui")
     healthGui.Name = "HealthGui"
     healthGui.Size = UDim2.new(5, 0, 1, 0)
     healthGui.StudsOffset = Vector3.new(0, 3, 0)
     healthGui.Adornee = hitbox
     healthGui.AlwaysOnTop = true
     healthGui.Parent = hitbox

     local healthLabel = Instance.new("TextLabel")
     healthLabel.Name = "HealthLabel"
     healthLabel.Size = UDim2.new(1, 0, 1, 0)
     healthLabel.BackgroundTransparency = 1
     healthLabel.TextColor3 = Color3.new(1, 1, 1)
     healthLabel.TextStrokeTransparency = 0.5
     healthLabel.TextScaled = true
     healthLabel.Parent = healthGui
    end

    hitbox.CFrame = head.CFrame * CFrame.new(0, 2, 0)

    local hum = v.Character:FindFirstChildOfClass("Humanoid")
    if hum then
     local hp = hum.Health
     local emojiGui = hitbox:FindFirstChild("Emoji")
     local healthGui = hitbox:FindFirstChild("HealthGui")

     if emojiGui and emojiGui:FindFirstChild("EmojiLabel") then
      local e = emojiGui.EmojiLabel
      if hp <= 35 then
       e.Text = "☠️"
       e.TextColor3 = Color3.fromRGB(255, 0, 0)
      else
       e.Text = "😂"
       e.TextColor3 = Color3.fromRGB(255, 255, 0)
      end
     end

     if healthGui and healthGui:FindFirstChild("HealthLabel") then
      healthGui.HealthLabel.Text = string.format("HP: %.0f", hp)
     end
    end
   end)
  end
 end
end)

-- Keybind toggle for ESP + follow
UserInputService.InputBegan:Connect(function(input, processed)
 if not processed and input.KeyCode == _G.ToggleKey then
  _G.Disabled = not _G.Disabled
  frame.Visible = not _G.Disabled
 end
end)
