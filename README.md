-- ============================================
-- OZ HUB - DELTA MOBILE
-- ============================================

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local SoundService = game:GetService("SoundService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ============================================
-- SISTEMA DE NOTIFICAÇÃO
-- ============================================
local NotificationHolder = nil
local notificacoesAtivas = {}
local rgbHue = 0
local rainbowConnection = nil

local function CreateNotificationHolder()
    if NotificationHolder then return end
    
    local gui = Instance.new("ScreenGui")
    gui.Name = "OzHubNotifications"
    gui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local holder = Instance.new("Frame")
    holder.Name = "NotificationHolder"
    holder.Parent = gui
    holder.Size = UDim2.new(0, 280, 0, 0)
    holder.Position = UDim2.new(0, 15, 1, -15)
    holder.AnchorPoint = Vector2.new(0, 1)
    holder.BackgroundTransparency = 1
    holder.AutomaticSize = Enum.AutomaticSize.Y
    holder.ClipsDescendants = false
    
    local layout = Instance.new("UIListLayout", holder)
    layout.Padding = UDim.new(0, 10)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.VerticalAlignment = Enum.VerticalAlignment.Bottom
    
    NotificationHolder = holder
end

local function GetRainbowColor(offset)
    local hue = (rgbHue + (offset or 0)) % 1
    return Color3.fromHSV(hue, 1, 1)
end

local function StartRainbowAnimation()
    if rainbowConnection then return end
    rainbowConnection = RunService.RenderStepped:Connect(function()
        rgbHue = (rgbHue + 0.003) % 1
        for _, notif in pairs(notificacoesAtivas) do
            if notif.stroke and notif.stroke.Parent then
                notif.stroke.Color = GetRainbowColor(notif.offset or 0)
            end
            if notif.progressBar and notif.progressBar.Parent then
                notif.progressBar.BackgroundColor3 = GetRainbowColor(notif.offset or 0)
            end
        end
    end)
end

local function StopRainbowAnimation()
    if #notificacoesAtivas == 0 and rainbowConnection then
        rainbowConnection:Disconnect()
        rainbowConnection = nil
    end
end

local function PlayNotificationSound()
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://71450094482101"
    sound.Volume = 0.25
    sound.Parent = SoundService
    sound:Play()
    sound.Ended:Connect(function()
        sound:Destroy()
    end)
end

local function AddNotification(title, content, duration)
    CreateNotificationHolder()
    
    if #notificacoesAtivas == 0 then
        StartRainbowAnimation()
    end
    
    PlayNotificationSound()
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 0)
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
    frame.BackgroundTransparency = 0.2
    frame.AutomaticSize = Enum.AutomaticSize.Y
    frame.BorderSizePixel = 0
    frame.ClipsDescendants = true
    frame.LayoutOrder = #notificacoesAtivas + 1
    frame.Parent = NotificationHolder
    frame.Transparency = 1
    
    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0, 12)
    
    local stroke = Instance.new("UIStroke", frame)
    stroke.Thickness = 1.5
    stroke.Color = GetRainbowColor(#notificacoesAtivas * 0.2)
    stroke.Transparency = 0.3
    
    local padding = Instance.new("UIPadding", frame)
    padding.PaddingLeft = UDim.new(0, 14)
    padding.PaddingRight = UDim.new(0, 14)
    padding.PaddingTop = UDim.new(0, 12)
    padding.PaddingBottom = UDim.new(0, 12)
    
    local titleLabel = Instance.new("TextLabel", frame)
    titleLabel.Text = title
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextSize = 14
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Size = UDim2.new(1, 0, 0, 20)
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.TextTruncate = Enum.TextTruncate.AtEnd
    
    local contentLabel = Instance.new("TextLabel", frame)
    contentLabel.Text = content
    contentLabel.Font = Enum.Font.Gotham
    contentLabel.TextSize = 12
    contentLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
    contentLabel.BackgroundTransparency = 1
    contentLabel.Size = UDim2.new(1, 0, 0, 18)
    contentLabel.TextXAlignment = Enum.TextXAlignment.Left
    contentLabel.TextTruncate = Enum.TextTruncate.AtEnd
    contentLabel.Position = UDim2.new(0, 0, 0, 22)
    
    frame.Size = UDim2.new(1, 0, 0, 52)
    
    local notifData = {
        frame = frame,
        stroke = stroke,
        offset = #notificacoesAtivas * 0.12,
        duration = duration or 5,
        progressBar = nil
    }
    table.insert(notificacoesAtivas, notifData)
    
    frame.Position = UDim2.new(1.2, 0, 0, 0)
    frame.Transparency = 1
    
    local tween1 = TweenService:Create(frame, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = UDim2.new(-0.02, 0, 0, 0),
        Transparency = 0
    })
    
    local tween2 = TweenService:Create(frame, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
        Position = UDim2.new(0, 0, 0, 0)
    })
    
    tween1:Play()
    tween1.Completed:Connect(function()
        tween2:Play()
    end)
    
    local progressBar = Instance.new("Frame", frame)
    progressBar.Name = "ProgressBar"
    progressBar.Size = UDim2.new(1, 0, 0, 2.5)
    progressBar.Position = UDim2.new(0, 0, 1, -2.5)
    progressBar.BackgroundColor3 = GetRainbowColor(notifData.offset)
    progressBar.BackgroundTransparency = 0.2
    progressBar.BorderSizePixel = 0
    
    local progressCorner = Instance.new("UICorner", progressBar)
    progressCorner.CornerRadius = UDim.new(0, 2)
    
    notifData.progressBar = progressBar
    
    local startTime = tick()
    local totalDuration = duration or 5
    
    local progressConnection
    progressConnection = RunService.RenderStepped:Connect(function()
        if not frame or not frame.Parent then
            if progressConnection then progressConnection:Disconnect() end
            return
        end
        
        local elapsed = tick() - startTime
        local percent = math.clamp(1 - (elapsed / totalDuration), 0, 1)
        
        if percent <= 0 then
            if progressConnection then progressConnection:Disconnect() end
        else
            progressBar.Size = UDim2.new(percent, 0, 0, 2.5)
            progressBar.BackgroundColor3 = GetRainbowColor(notifData.offset)
        end
    end)
    
    task.delay(duration or 5, function()
        for i, data in pairs(notificacoesAtivas) do
            if data.frame == frame then
                table.remove(notificacoesAtivas, i)
                break
            end
        end
        
        StopRainbowAnimation()
        
        if progressConnection then progressConnection:Disconnect() end
        
        local exitTween = TweenService:Create(frame, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
            Position = UDim2.new(-1.2, 0, 0, 0),
            Transparency = 1
        })
        exitTween:Play()
        
        exitTween.Completed:Connect(function()
            frame:Destroy()
        end)
    end)
end

-- ============================================
--  CONFIGURAÇÕES CENTRALIZADAS
-- ============================================
local Settings = {
    -- Aimbot
    AimbotEnabled = false,
    SelectedAimPart = "Head",
    AimbotFOV = 90,
    AimbotSmoothness = 5,
    IgnoreWalls = false,
    LockCamEnabled = false,
    LockUIActive = false,
    
    -- ESP
    ESPHighlightEnabled = false,
    ESPMM2HighlightEnabled = false,
    HideNameEnabled = false,
    ShowFPSEnabled = false,
    
    -- Movimento
    SpeedToggleEnabled = false,
    SpeedValue = 16,
    FOVCamToggleEnabled = false,
    FovCamValue = 70,
    infJumpEnabled = false,
    NoClipEnabled = false,
    flying = false,
    FlySpeed = 50,
    SpinbotEnabled = false,
    
    -- AutoFarm
    AutoFarmEnabled = false,
    AutoFarmMoving = false,
    AutoFarmAtivouNoClip = false,
    
    -- Onlines / Interações
    backpackActive = false,
    headsitActive = false,
    spectateActive = false,
    flingOPActive = false,
    
    -- MM2
    killAssassinActive = false,
    killSheriffActive = false,
    killAssassinPos = nil,
    killSheriffPos = nil,
    killAssassinConnection = nil,
    killSheriffConnection = nil,
    
    -- Visuais
    headlessActive = false,
    korbloxActive = false,
    originalHead = nil,
    originalLeftLeg = nil,
    
    -- Camera
    LockUI = nil,
    MiraUI = nil,
    
    -- Emotes e Animações
    emoteSpeed = 1,
    loopEmote = false,
    currentEmoteTrack = nil,
    selectedAnimationPack = nil,
    
    -- Teleport
    savedPosition = nil,
    lastPositionTP = nil,
    
    -- Caches
    lastCoinCacheUpdate = 0,
    lastFarmTime = 0,
    lastAimbotTime = 0,
    lastESPUpdate = 0,
    lastDropdownUpdate = 0,
    lastPositionsCount = 0,
    
    -- Performance
    PerformanceMode = false,
    OriginalBrightness = nil,
    OriginalGlobalShadows = nil,
    
    -- Seleção de player
    currentSelectedPlayer = nil,
}

-- ============================================
--  VARIÁVEIS GLOBAIS
-- ============================================

local lastPositions = {}
local coinCache = {}
local playerCache = {}
local ESPHighlights = {}
local RaycastCache = {}
local FOVCircle = nil
local FOVCircleFrame = nil
local bodyVelocity = nil
local bodyGyro = nil
local AutoFarmTween = nil
local currentEmoteTrack = nil
local posicaoOriginal = nil
local savedPosition = nil
local lastPositionTP = nil
local originalHead = nil
local originalLeftLeg = nil
local selectedAnimationPack = nil
local currentSelectedPlayer = nil

local NoClipToggle = nil
local AutoFarmToggleElement = nil
local ESPNormalToggle = nil
local ESPMM2Toggle = nil
local LockUI = nil
local MiraUI = nil
local playerDropdown = nil
local infoSection = nil
local infoParagraph = nil

local jumpConnection = nil

-- ============================================
--  FUNÇÕES UTILITÁRIAS
-- ============================================

local function GetChar() return player.Character end
local function GetHRP(char) 
    char = char or GetChar() 
    return char and char:FindFirstChild("HumanoidRootPart") 
end
local function GetHumanoid(char) 
    char = char or GetChar() 
    return char and char:FindFirstChild("Humanoid") 
end

local function getHeadshot(plr)
    return Players:GetUserThumbnailAsync(plr.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420)
end

-- ============================================
-- NOCLIP ROBUSTO
-- ============================================
local NoClipConnection = nil

local function aplicarNoClip()
    if not Settings.NoClipEnabled then
        if NoClipConnection then
            NoClipConnection:Disconnect()
            NoClipConnection = nil
        end
        return
    end
    
    if NoClipConnection then
        NoClipConnection:Disconnect()
        NoClipConnection = nil
    end
    
    NoClipConnection = RunService.Stepped:Connect(function()
        if not Settings.NoClipEnabled then
            if NoClipConnection then
                NoClipConnection:Disconnect()
                NoClipConnection = nil
            end
            return
        end
        
        local char = GetChar()
        if not char then return end
        
        pcall(function()
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
    end)
end

local function reverterNoClip()
    if NoClipConnection then
        NoClipConnection:Disconnect()
        NoClipConnection = nil
    end
    
    local char = GetChar()
    if char then
        pcall(function()
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end)
    end
end

-- ============================================
--  LOOP MANAGER
-- ============================================
local LoopManager = { tasks = {}, conn = nil }

function LoopManager:Add(name, callback, frequency)
    if self.tasks[name] then return false end
    self.tasks[name] = { callback = callback, frequency = frequency or 0, lastRun = 0 }
    if not self.conn and next(self.tasks) then
        self.conn = RunService.Heartbeat:Connect(function(dt)
            local now = tick()
            for taskName, task in pairs(self.tasks) do
                if task.frequency == 0 or (now - task.lastRun) >= task.frequency then
                    task.lastRun = now
                    local success, err = pcall(task.callback, dt)
                    if not success then 
                        warn("[LoopManager] Erro em", taskName, err) 
                    end
                end
            end
        end)
    end
    return true
end

function LoopManager:Remove(name)
    self.tasks[name] = nil
    if not next(self.tasks) and self.conn then
        self.conn:Disconnect()
        self.conn = nil
    end
end

function LoopManager:Clear()
    self.tasks = {}
    if self.conn then
        self.conn:Disconnect()
        self.conn = nil
    end
end

-- ============================================
--  EVENT MANAGER
-- ============================================
local EventManager = { events = {} }

function EventManager:Add(name, connection)
    if self.events[name] then 
        pcall(function() self.events[name]:Disconnect() end)
    end
    self.events[name] = connection
end

function EventManager:Clear()
    for _, conn in pairs(self.events) do
        pcall(function() conn:Disconnect() end)
    end
    self.events = {}
end

-- ============================================
--  AIMBOT
-- ============================================

local function updatePlayerCache()
    playerCache = {}
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then table.insert(playerCache, plr) end
    end
end

local function CanSeePlayer(target)
    local myChar = GetChar()
    if not myChar or not myChar:FindFirstChild("Head") then return false end
    if not target.Character or not target.Character:FindFirstChild("Head") then return false end
    
    local origin = camera.CFrame.Position
    local targetPos = target.Character.Head.Position
    local direction = (targetPos - origin).Unit * 1000
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {myChar}
    
    local result = workspace:Raycast(origin, direction, raycastParams)
    if result then return result.Instance:IsDescendantOf(target.Character) end
    return true
end

local function UpdateFOVCircle()
    if not Settings.AimbotEnabled then
        if FOVCircle then
            FOVCircle:Destroy()
            FOVCircle = nil
            FOVCircleFrame = nil
        end
        return
    end
    
    if not FOVCircle then
        local gui = Instance.new("ScreenGui")
        gui.Name = "FOVCircle"
        gui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
        gui.ResetOnSpawn = false
        gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        gui.DisplayOrder = 999
        gui.IgnoreGuiInset = true
        
        local circle = Instance.new("Frame")
        circle.Parent = gui
        circle.BackgroundTransparency = 1
        circle.BorderSizePixel = 0
        
        local stroke = Instance.new("UIStroke")
        stroke.Parent = circle
        stroke.Color = Color3.fromRGB(255, 255, 255)
        stroke.Thickness = 0.3
        stroke.Transparency = 0
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(1, 0)
        corner.Parent = circle
        
        FOVCircle = gui
        FOVCircleFrame = circle
    end
    
    FOVCircleFrame.Size = UDim2.new(0, Settings.AimbotFOV * 2, 0, Settings.AimbotFOV * 2)
    FOVCircleFrame.Position = UDim2.new(0.5, -Settings.AimbotFOV, 0.5, -Settings.AimbotFOV)
end

local function GetClosestPlayerWithFOV(ignoreWalls)
    local closestPlayer = nil
    local bestScore = math.huge
    local myChar = GetChar()
    if not myChar then return nil end
    
    local myHead = myChar:FindFirstChild("Head")
    if not myHead then return nil end
    local myPos = myHead.Position
    local cameraPos = camera.CFrame.Position
    
    updatePlayerCache()
    
    for _, otherPlayer in pairs(playerCache) do
        local char = otherPlayer.Character
        if not char then continue end
        
        local aimPart = Settings.SelectedAimPart == "Head" and char:FindFirstChild("Head") or GetHRP(char)
        if not aimPart then continue end
        
        local humanoid = GetHumanoid(char)
        if not humanoid or humanoid.Health <= 0 then continue end
        
        if ignoreWalls and not CanSeePlayer(otherPlayer) then continue end
        
        local targetPos = aimPart.Position
        local realDist = (targetPos - myPos).Magnitude
        if realDist > 300 then continue end
        
        if lastPositions[otherPlayer] then
            local lastPos, lastTime = lastPositions[otherPlayer].pos, lastPositions[otherPlayer].time
            local dt = tick() - lastTime
            if dt > 0 and dt < 0.3 then
                local velocity = (targetPos - lastPos) / dt
                targetPos = targetPos + (velocity * 0.18)
            end
        end
        
        lastPositions[otherPlayer] = {pos = aimPart.Position, time = tick()}
        
        local screenPoint = camera:WorldToViewportPoint(targetPos)
        if screenPoint.Z <= 0 then continue end
        
        local centerX = camera.ViewportSize.X / 2
        local centerY = camera.ViewportSize.Y / 2
        local centerDist = math.sqrt((screenPoint.X - centerX)^2 + (screenPoint.Y - centerY)^2)
        
        if centerDist > Settings.AimbotFOV then continue end
        
        local score = centerDist + (realDist * 0.2)
        
        if score < bestScore then
            bestScore = score
            closestPlayer = otherPlayer
        end
    end
    
    return closestPlayer
end

local function AimbotTask()
    if not Settings.AimbotEnabled then return end
    if not camera or not player.Character then return end
    
    local target = GetClosestPlayerWithFOV(Settings.IgnoreWalls)
    if not target or not target.Character then return end
    
    local aimPart = Settings.SelectedAimPart == "Head" and target.Character:FindFirstChild("Head") or GetHRP(target.Character)
    if not aimPart then return end
    
    local targetPos = aimPart.Position
    
    if lastPositions[target] then
        local lastPos, lastTime = lastPositions[target].pos, lastPositions[target].time
        local dt = tick() - lastTime
        if dt > 0 and dt < 0.3 then
            local velocity = (targetPos - lastPos) / dt
            targetPos = targetPos + (velocity * 0.2)
        end
    end
    
    local targetCF = CFrame.lookAt(camera.CFrame.Position, targetPos)
    
    if Settings.AimbotSmoothness == 0 then
        camera.CFrame = targetCF
    else
        local smoothFactor = 1 / (Settings.AimbotSmoothness * 1.2)
        camera.CFrame = camera.CFrame:Lerp(targetCF, smoothFactor)
    end
end

-- ============================================
--  ESP (CONTORNO RGB)
-- ============================================

local function getPapel(jogador)
    local char = jogador.Character
    if not char then return "Inocente" end
    if char:FindFirstChild("Knife") or char:FindFirstChild("FadeKnife") then return "Assassino" end
    if char:FindFirstChild("Gun") or char:FindFirstChild("Pistol") or char:FindFirstChild("Revolver") then return "Xerife" end
    if jogador.Backpack then
        if jogador.Backpack:FindFirstChild("Knife") or jogador.Backpack:FindFirstChild("FadeKnife") then return "Assassino" end
        if jogador.Backpack:FindFirstChild("Gun") or jogador.Backpack:FindFirstChild("Pistol") or jogador.Backpack:FindFirstChild("Revolver") then return "Xerife" end
    end
    return "Inocente"
end

local function limparESPHighlight()
    for player, data in pairs(ESPHighlights) do
        pcall(function()
            if data.Highlight then data.Highlight:Destroy() end
            if data.Billboard then data.Billboard:Destroy() end
        end)
    end
    ESPHighlights = {}
end

local ESPRainbowConnection = nil

local function atualizarESPHighlight()
    if not (Settings.ESPHighlightEnabled or Settings.ESPMM2HighlightEnabled) then
        if next(ESPHighlights) then limparESPHighlight() end
        return
    end
    
    local myChar = GetChar()
    if not myChar then return end
    local myHRP = GetHRP(myChar)
    if not myHRP then return end
    local myPos = myHRP.Position
    
    for _, plr in pairs(Players:GetPlayers()) do
        if plr == player then continue end
        
        local char = plr.Character
        if not char then
            if ESPHighlights[plr] then
                pcall(function()
                    if ESPHighlights[plr].Highlight then ESPHighlights[plr].Highlight:Destroy() end
                    if ESPHighlights[plr].Billboard then ESPHighlights[plr].Billboard:Destroy() end
                end)
                ESPHighlights[plr] = nil
            end
            continue
        end
        
        local humanoid = GetHumanoid(char)
        local root = GetHRP(char)
        
        if not humanoid or humanoid.Health <= 0 or not root then
            if ESPHighlights[plr] then
                pcall(function()
                    if ESPHighlights[plr].Highlight then ESPHighlights[plr].Highlight:Destroy() end
                    if ESPHighlights[plr].Billboard then ESPHighlights[plr].Billboard:Destroy() end
                end)
                ESPHighlights[plr] = nil
            end
            continue
        end
        
        local distancia = (root.Position - myPos).Magnitude
        if distancia > 250 then
            if ESPHighlights[plr] then
                pcall(function()
                    if ESPHighlights[plr].Highlight then ESPHighlights[plr].Highlight:Destroy() end
                    if ESPHighlights[plr].Billboard then ESPHighlights[plr].Billboard:Destroy() end
                end)
                ESPHighlights[plr] = nil
            end
            continue
        end
        
        local cor = Color3.fromRGB(255, 0, 0)
        if Settings.ESPMM2HighlightEnabled then
            local papel = getPapel(plr)
            if papel == "Assassino" then
                cor = Color3.fromRGB(255, 0, 0)
            elseif papel == "Xerife" then
                cor = Color3.fromRGB(0, 100, 255)
            else
                cor = Color3.fromRGB(0, 255, 0)
            end
        end
        
        if not ESPHighlights[plr] then
            local h = Instance.new("Highlight")
            h.Parent = char
            h.FillTransparency = 1
            h.OutlineTransparency = 0
            h.OutlineColor = cor
            h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            
            local billboard = Instance.new("BillboardGui")
            local head = char:FindFirstChild("Head")
            billboard.Parent = head or char
            billboard.Size = UDim2.new(0, 100, 0, 30)
            billboard.StudsOffset = Vector3.new(0, 2.5, 0)
            billboard.AlwaysOnTop = true
            
            local textLabel = Instance.new("TextLabel", billboard)
            textLabel.Size = UDim2.new(1, 0, 1, 0)
            textLabel.BackgroundTransparency = 1
            textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            textLabel.Text = plr.Name .. " [" .. math.floor(distancia) .. "m]"
            textLabel.Font = Enum.Font.GothamBold
            textLabel.TextSize = 8
            
            ESPHighlights[plr] = {Highlight = h, Billboard = billboard, TextLabel = textLabel}
        else
            local data = ESPHighlights[plr]
            if data.Highlight and Settings.ESPMM2HighlightEnabled then
                data.Highlight.OutlineColor = cor
            end
            if data.TextLabel then
                data.TextLabel.Text = plr.Name .. " [" .. math.floor(distancia) .. "m]"
            end
        end
    end
end

local function iniciarESPHighlight()
    LoopManager:Add("ESP", atualizarESPHighlight, 1.2)
    
    if ESPRainbowConnection then ESPRainbowConnection:Disconnect() end
    ESPRainbowConnection = RunService.RenderStepped:Connect(function()
        if not Settings.ESPHighlightEnabled or Settings.ESPMM2HighlightEnabled then
            return
        end
        local hue = (tick() * 0.3) % 1
        local rainbowColor = Color3.fromHSV(hue, 1, 1)
        for _, data in pairs(ESPHighlights) do
            if data.Highlight then
                data.Highlight.OutlineColor = rainbowColor
            end
        end
    end)
end

local function pararESPHighlight()
    LoopManager:Remove("ESP")
    if ESPRainbowConnection then
        ESPRainbowConnection:Disconnect()
        ESPRainbowConnection = nil
    end
    limparESPHighlight()
end

-- ============================================
--  MOVIMENTO
-- ============================================

local function aplicarSpeed()
    if not Settings.SpeedToggleEnabled then return end
    local char = GetChar()
    if char then
        local humanoid = GetHumanoid(char)
        if humanoid then
            humanoid.WalkSpeed = Settings.SpeedValue
        end
    end
end

local function reverterSpeed()
    local char = GetChar()
    if char then
        local humanoid = GetHumanoid(char)
        if humanoid then
            humanoid.WalkSpeed = 16
        end
    end
end

local function aplicarFov()
    if not Settings.FOVCamToggleEnabled then return end
    if camera then
        camera.FieldOfView = Settings.FovCamValue
    end
end

local function reverterFov()
    if camera then
        camera.FieldOfView = 70
    end
end

local function startFly()
    local char = GetChar()
    if not char then
        AddNotification("OZ HUB", "Personagem nao encontrado", 3)
        return
    end
    
    local humanoid = GetHumanoid(char)
    local hrp = GetHRP(char)
    
    if not humanoid or not hrp then
        AddNotification("OZ HUB", "Humanoid ou HRP nao encontrado", 3)
        return
    end
    
    if bodyVelocity then bodyVelocity:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
    
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Name = "FlyVelocity"
    bodyVelocity.MaxForce = Vector3.new(999999, 999999, 999999)
    bodyVelocity.Parent = hrp
    
    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.Name = "FlyGyro"
    bodyGyro.MaxTorque = Vector3.new(999999, 999999, 999999)
    bodyGyro.Parent = hrp
    
    humanoid.PlatformStand = true
    
    LoopManager:Add("FlyUpdate", function()
        if not Settings.flying then return end
        if not camera or not player.Character then return end
        
        local char = GetChar()
        local hrp = GetHRP(char)
        local humanoid = GetHumanoid(char)
        
        if not char or not humanoid or not hrp then
            Settings.flying = false
            return
        end
        
        local moveDir = humanoid.MoveDirection
        local velocity = moveDir * Settings.FlySpeed
        
        if moveDir.Magnitude > 0 then
            local lookY = camera.CFrame.LookVector.Y
            local forwardDot = moveDir:Dot(camera.CFrame.LookVector)
            local directionFix = (forwardDot >= 0) and 1 or -1
            velocity = velocity + Vector3.new(0, lookY * Settings.FlySpeed * directionFix, 0)
        end
        
        if bodyVelocity then bodyVelocity.Velocity = velocity end
        if bodyGyro then bodyGyro.CFrame = camera.CFrame end
    end, 0)
end

local function StopFly()
    Settings.flying = false
    local char = GetChar()
    if char then
        local humanoid = GetHumanoid(char)
        if humanoid then
            humanoid.PlatformStand = false
        end
    end
    if bodyVelocity then bodyVelocity:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
    bodyVelocity, bodyGyro = nil, nil
    LoopManager:Remove("FlyUpdate")
end

local function toggleFly(state)
    Settings.flying = state
    if state then
        startFly()
    else
        StopFly()
    end
end

local function SpinbotTask()
    if not Settings.SpinbotEnabled then return end
    local char = GetChar()
    if not char then return end
    local hrp = GetHRP(char)
    if hrp then
        hrp.RotVelocity = Vector3.new(0, 150, 0)
    end
end

local function toggleSpinbot(state)
    Settings.SpinbotEnabled = state
    if state then
        LoopManager:Add("Spinbot", SpinbotTask, 0)
    else
        LoopManager:Remove("Spinbot")
        local char = GetChar()
        if char then
            local hrp = GetHRP(char)
            if hrp then
                hrp.RotVelocity = Vector3.new(0, 0, 0)
            end
        end
    end
end

-- ============================================
--  LOCK CAM
-- ============================================

local function CreateShiftLockButton()
    if LockUI then LockUI:Destroy() end
    
    local gui = Instance.new("ScreenGui")
    gui.Name = "ShiftLockButton"
    gui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 999
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local button = Instance.new("ImageButton")
    button.Parent = gui
    button.Name = "ShiftLockButton"
    button.BackgroundTransparency = 1
    button.Size = UDim2.new(0, 70, 0, 70)
    button.Position = UDim2.new(1, -200, 0.5, -5)
    button.Active = true
    button.Draggable = false
    button.BorderSizePixel = 0
    button.AutoButtonColor = false
    button.ZIndex = 2
    
    local function UpdateButtonState(active)
        if active then
            button.Image = "rbxassetid://139211296111194"
        else
            button.Image = "rbxassetid://16812589014"
        end
        if MiraUI then
            MiraUI.Enabled = active
        end
        Settings.LockUIActive = active
    end
    
    button.MouseButton1Click:Connect(function()
        local novoEstado = not Settings.LockUIActive
        UpdateButtonState(novoEstado)
    end)
    
    UpdateButtonState(false)
    LockUI = gui
end

local function CriarMira()
    if MiraUI then MiraUI:Destroy() end
    
    local gui = Instance.new("ScreenGui")
    gui.Name = "MiraCentral"
    gui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 1000
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Enabled = false
    
    local mira = Instance.new("ImageLabel")
    mira.Parent = gui
    mira.BackgroundTransparency = 1
    mira.Size = UDim2.new(0, 23, 0, 23)
    mira.Position = UDim2.new(0.5, -11, 0.5, -12)
    mira.Image = "rbxassetid://17404114716"
    
    MiraUI = gui
end

local function LockCamTask()
    if Settings.LockCamEnabled and Settings.LockUIActive then
        local character = GetChar()
        if character and GetHRP(character) then
            local rootPart = GetHRP(character)
            local humanoid = GetHumanoid(character)
            if humanoid then
                humanoid.CameraOffset = Vector3.new(1.5, 0, 0)
            end
            local cameraDirection = camera.CFrame.LookVector
            cameraDirection = Vector3.new(cameraDirection.X, 0, cameraDirection.Z).Unit
            if cameraDirection.Magnitude > 0 then
                rootPart.CFrame = CFrame.lookAt(rootPart.Position, rootPart.Position + cameraDirection)
            end
        end
    else
        local char = GetChar()
        if char then
            local humanoid = GetHumanoid(char)
            if humanoid then
                humanoid.CameraOffset = Vector3.new(0, 0, 0)
            end
        end
    end
end

-- ============================================
--  AUTOFARM
-- ============================================

local function atualizarCacheMoedas()
    if tick() - Settings.lastCoinCacheUpdate < 2 then return end
    Settings.lastCoinCacheUpdate = tick()
    coinCache = {}
    
    local coinFolder = workspace:FindFirstChild("Coins") or workspace:FindFirstChild("Drops") or workspace:FindFirstChild("Money")
    if coinFolder then
        for _, obj in pairs(coinFolder:GetChildren()) do
            if obj:IsA("BasePart") and obj.Transparency < 0.8 then
                local nome = obj.Name:lower()
                if nome:find("coin") or nome:find("gold") or nome:find("money") then
                    table.insert(coinCache, obj)
                end
            end
        end
    else
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Transparency < 0.8 then
                local nome = obj.Name:lower()
                if nome:find("coin") or nome:find("gold") or nome:find("money") then
                    table.insert(coinCache, obj)
                end
            end
        end
    end
end

workspace.DescendantAdded:Connect(function() coinCache = {} end)
workspace.DescendantRemoving:Connect(function() coinCache = {} end)

local function pararAutoFarmMovimento()
    if AutoFarmTween and AutoFarmTween.PlaybackState == Enum.PlaybackState.Playing then
        AutoFarmTween:Cancel()
    end
    Settings.AutoFarmMoving = false
    AutoFarmTween = nil
    
    if Settings.AutoFarmAtivouNoClip then
	    pcall(function()
	        local char = GetChar()
	        if char then
	            for _, part in pairs(char:GetDescendants()) do
	                if part:IsA("BasePart") then
	                    part.CanCollide = true
	                end
	            end
	        end
	    end)
	    Settings.AutoFarmAtivouNoClip = false
	end
end

local function moverParaMoeda(moeda)
    local char = GetChar()
    if not char then return false end
    local hrp = GetHRP(char)
    if not hrp then return false end
    
    if not Settings.NoClipEnabled then
	    pcall(function()
	        local char = GetChar()
	        if char then
	            for _, part in pairs(char:GetDescendants()) do
	                if part:IsA("BasePart") then
	                    part.CanCollide = false
	                end
	            end
	        end
	    end)
	    Settings.AutoFarmAtivouNoClip = true
	end
    
    local targetPos = moeda.Position + Vector3.new(0, 2, 0)
    local distancia = (targetPos - hrp.Position).Magnitude
    local duracao = math.max(distancia / 45, 0.2)
    
    AutoFarmTween = TweenService:Create(hrp, TweenInfo.new(duracao, Enum.EasingStyle.Linear), {CFrame = CFrame.new(targetPos)})
    Settings.AutoFarmMoving = true
    AutoFarmTween:Play()
    AutoFarmTween.Completed:Connect(function()
        task.wait(0.2)
        Settings.AutoFarmMoving = false
    end)
    
    return true
end

local function getNearestCoinNatural()
    local myHRP = GetHRP()
    if not myHRP then return nil end
    
    atualizarCacheMoedas()
    
    local nearestCoin = nil
    local nearestDist = 200
    
    for _, obj in pairs(coinCache) do
        if obj and obj.Parent then
            local dist = (obj.Position - myHRP.Position).Magnitude
            if dist < nearestDist then
                nearestDist = dist
                nearestCoin = obj
            end
        end
    end
    
    return nearestCoin
end

local function AutoFarmTaskNatural()
    if not Settings.AutoFarmEnabled then return end
    if Settings.AutoFarmMoving then return end
    if not player.Character then return end
    
    local moeda = getNearestCoinNatural()
    if moeda then moverParaMoeda(moeda) end
end

local function AutoFarmToggle(Value)
    Settings.AutoFarmEnabled = Value
    if Value then
        if NoClipToggle then NoClipToggle:Lock() end
        LoopManager:Add("AutoFarm", AutoFarmTaskNatural, 1.2)
        AddNotification("OZ HUB", "AutoFarm ativado", 3)
    else
        if NoClipToggle then NoClipToggle:Unlock() end
        LoopManager:Remove("AutoFarm")
        pararAutoFarmMovimento()
        AddNotification("OZ HUB", "AutoFarm desativado", 3)
    end
end

-- ============================================
--  ONLINES FUNCTIONS
-- ============================================

local function stopAll()
    if Settings.backpackActive then
        Settings.backpackActive = false
        LoopManager:Remove("Backpack")
    end
    if Settings.headsitActive then
        Settings.headsitActive = false
        LoopManager:Remove("Headsit")
    end
    if Settings.spectateActive then
        Settings.spectateActive = false
        local myChar = GetChar()
        if myChar and myChar:FindFirstChild("Humanoid") then
            camera.CameraSubject = myChar.Humanoid
            camera.CameraType = Enum.CameraType.Custom
        end
    end
    if Settings.flingOPActive then
        Settings.flingOPActive = false
        LoopManager:Remove("FlingOP")
        local meuHRP = GetHRP()
        if meuHRP then
            meuHRP.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            meuHRP.RotVelocity = Vector3.new(0, 0, 0)
            meuHRP.Velocity = Vector3.new(0, 0, 0)
            if posicaoOriginal then
                meuHRP.CFrame = posicaoOriginal
                posicaoOriginal = nil
            end
        end
    end
    local char = GetChar()
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.Sit = false
    end
end

-- ============================================
--  HEADLESS E KORBLOX
-- ============================================

local function toggleHeadless(state)
    local char = GetChar()
    if not char then return end
    
    local head = char:FindFirstChild("Head")
    if not head then return end
    
    local hairKeywords = {"hair", "cabelo", "wig", "cabel", "hairstyle", "bangs", "ponytail", "braid", "haircut"}
    
    local function isHair(accessoryName)
        local lowerName = accessoryName:lower()
        for _, keyword in pairs(hairKeywords) do
            if lowerName:find(keyword) then return true end
        end
        return false
    end
    
    if state then
        originalHead = head:Clone()
        originalHead.Parent = nil
        head.Transparency = 1
        head.CanCollide = false
        head.Massless = true
        
        local face = head:FindFirstChildOfClass("Decal")
        if face then face.Transparency = 1 end
        
        for _, accessory in pairs(char:GetChildren()) do
            if accessory:IsA("Accessory") then
                local handle = accessory:FindFirstChild("Handle")
                local weld = handle and handle:FindFirstChildOfClass("Weld")
                if isHair(accessory.Name) then
                    if handle then handle.Transparency = 0 end
                else
                    if weld and weld.Part1 == head then
                        if handle then handle.Transparency = 1 end
                    end
                end
            end
        end
    else
        head.Transparency = 0
        head.CanCollide = true
        head.Massless = false
        
        local face = head:FindFirstChildOfClass("Decal")
        if face then face.Transparency = 0 end
        
        for _, accessory in pairs(char:GetChildren()) do
            if accessory:IsA("Accessory") then
                local handle = accessory:FindFirstChild("Handle")
                if handle then handle.Transparency = 0 end
            end
        end
        
        if originalHead then originalHead:Destroy() end
        originalHead = nil
    end
end

local function toggleKorblox(state)
    local char = GetChar()
    if not char then return end
    
    local parts = {"LeftUpperLeg", "LeftLowerLeg", "LeftFoot"}
    
    if state then
        originalLeftLeg = {}
        for _, partName in pairs(parts) do
            local part = char:FindFirstChild(partName)
            if part then
                originalLeftLeg[partName] = part.Transparency
                part.Transparency = 1
                part.CanCollide = false
                part.Massless = true
            end
        end
    else
        for _, partName in pairs(parts) do
            local part = char:FindFirstChild(partName)
            if part then
                part.Transparency = 0
                part.CanCollide = true
                part.Massless = false
            end
        end
        originalLeftLeg = nil
    end
end

-- ============================================
-- PAINEL DE INFORMAÇÕES (FPS, PING, PLAYERS)
-- ============================================
local InfoPanel = nil
local InfoPanelConnection = nil
local InfoPanelDragging = false
local InfoPanelDragStart = nil
local InfoPanelStartPos = nil
local InfoPanelRainbowConnection = nil
local InfoPanelRgbHue = 0

local function CreateInfoPanel()
    if InfoPanel then InfoPanel:Destroy() end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "OzInfoPanel"
    screenGui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder = 1000
    screenGui.IgnoreGuiInset = true
    
    local frame = Instance.new("Frame")
    frame.Parent = screenGui
    frame.Size = UDim2.new(0, 190, 0, 28)
    frame.Position = UDim2.new(0, 20, 0.5, -14)
    frame.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
    frame.BackgroundTransparency = 0.15
    frame.BorderSizePixel = 0
    frame.Active = true
    frame.Draggable = false
    frame.ClipsDescendants = true
    
    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0, 8)
    
    local stroke = Instance.new("UIStroke", frame)
    stroke.Thickness = 1.5
    stroke.Color = Color3.fromRGB(255, 100, 100)
    stroke.Transparency = 0.2
    
    local contentFrame = Instance.new("Frame", frame)
    contentFrame.Name = "Content"
    contentFrame.Size = UDim2.new(1, 0, 1, 0)
    contentFrame.BackgroundTransparency = 1
    
    local layout = Instance.new("UIListLayout", contentFrame)
    layout.FillDirection = Enum.FillDirection.Horizontal
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.VerticalAlignment = Enum.VerticalAlignment.Center
    layout.Padding = UDim.new(0, 6)
    
    local fpsLabel = Instance.new("TextLabel", contentFrame)
    fpsLabel.Name = "FPS"
    fpsLabel.Text = "FPS: 60"
    fpsLabel.Font = Enum.Font.GothamSemibold
    fpsLabel.TextSize = 12
    fpsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    fpsLabel.BackgroundTransparency = 1
    fpsLabel.Size = UDim2.new(0, 50, 1, 0)
    
    local sep1 = Instance.new("TextLabel", contentFrame)
    sep1.Text = "|"
    sep1.Font = Enum.Font.Gotham
    sep1.TextSize = 11
    sep1.TextColor3 = Color3.fromRGB(80, 80, 90)
    sep1.BackgroundTransparency = 1
    sep1.Size = UDim2.new(0, 10, 1, 0)
    
    local pingLabel = Instance.new("TextLabel", contentFrame)
    pingLabel.Name = "Ping"
    pingLabel.Text = "Ping: 0ms"
    pingLabel.Font = Enum.Font.GothamSemibold
    pingLabel.TextSize = 12
    pingLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    pingLabel.BackgroundTransparency = 1
    pingLabel.Size = UDim2.new(0, 55, 1, 0)
    
    local sep2 = Instance.new("TextLabel", contentFrame)
    sep2.Text = "|"
    sep2.Font = Enum.Font.Gotham
    sep2.TextSize = 11
    sep2.TextColor3 = Color3.fromRGB(80, 80, 90)
    sep2.BackgroundTransparency = 1
    sep2.Size = UDim2.new(0, 10, 1, 0)
    
    local playersLabel = Instance.new("TextLabel", contentFrame)
    playersLabel.Name = "Players"
    playersLabel.Text = "Players: 0"
    playersLabel.Font = Enum.Font.GothamSemibold
    playersLabel.TextSize = 12
    playersLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    playersLabel.BackgroundTransparency = 1
    playersLabel.Size = UDim2.new(0, 55, 1, 0)
    
    -- Ajuste de padding interno para centralizar melhor
    local padding = Instance.new("UIPadding", contentFrame)
    padding.PaddingLeft = UDim.new(0, 4)
    padding.PaddingRight = UDim.new(0, 4)
    
    if InfoPanelRainbowConnection then InfoPanelRainbowConnection:Disconnect() end
    InfoPanelRainbowConnection = RunService.RenderStepped:Connect(function()
        if not frame or not frame.Parent then
            if InfoPanelRainbowConnection then InfoPanelRainbowConnection:Disconnect() end
            InfoPanelRainbowConnection = nil
            return
        end
        InfoPanelRgbHue = (InfoPanelRgbHue + 0.005) % 1
        stroke.Color = Color3.fromHSV(InfoPanelRgbHue, 1, 1)
    end)
    
    local dragging = false
    local dragStart = nil
    local startPos = nil
    
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                local delta = input.Position - dragStart
                frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end
    end)
    
    InfoPanel = screenGui
    
    local frameCount = 0
    local lastTime = tick()
    
    if InfoPanelConnection then InfoPanelConnection:Disconnect() end
    InfoPanelConnection = RunService.RenderStepped:Connect(function()
        if not Settings.ShowFPSEnabled then
            if InfoPanelConnection then InfoPanelConnection:Disconnect() end
            InfoPanelConnection = nil
            return
        end
        
        frameCount = frameCount + 1
        local now = tick()
        if now - lastTime >= 1 then
            local fps = frameCount
            frameCount = 0
            lastTime = now
            
            local fpsColor = fps >= 50 and Color3.fromRGB(0, 255, 0) or (fps >= 30 and Color3.fromRGB(255, 200, 0) or Color3.fromRGB(255, 50, 50))
            fpsLabel.Text = "FPS: " .. fps
            fpsLabel.TextColor3 = fpsColor
            
            local ping = player:GetNetworkPing()
            local lastPing = math.floor(ping * 1000)
            pingLabel.Text = "Ping: " .. lastPing .. "ms"
            pingLabel.TextColor3 = lastPing <= 100 and Color3.fromRGB(0, 255, 0) or (lastPing <= 200 and Color3.fromRGB(255, 200, 0) or Color3.fromRGB(255, 50, 50))
            
            local playerCount = #Players:GetPlayers()
            playersLabel.Text = "Players: " .. playerCount
        end
    end)
end

-- ============================================
--  ANIMAÇÕES E EMOTES
-- ============================================

local ANIMATION_PACKS = {
    Pirate = { Idle1 = "750781874", Idle2 = "750782770", Walk = "750785693", Run = "750783738", Jump = "750782230", Fall = "750780242", Climb = "750779899", Swim = "750784579", SwimIdle = "750785176" },
    Cartoony = { Idle1 = "10921071918", Idle2 = "10921072875", Walk = "10921082452", Run = "10921076136", Jump = "10921078135", Fall = "10921077030", Climb = "10921070953", Swim = "10921079380", SwimIdle = "10921081059" },
    Hero = { Idle1 = "92080889861410", Idle2 = "74451233229259", Walk = "110358958299415", Run = "117333533048078", Jump = "119846112151352", Fall = "129773241321032", Climb = "134630013742019", Swim = "132697394189921", SwimIdle = "79090109939093" },
    Knight = { Idle1 = "10921117521", Idle2 = "10921118894", Walk = "10921127095", Run = "10921121197", Jump = "10921123517", Fall = "10921122579", Climb = "10921116196", Swim = "10921125160", SwimIdle = "10921125935" },
    SuperHero = { Idle1 = "10921288909", Idle2 = "10921290167", Walk = "10921298616", Run = "10921291831", Jump = "10921294559", Fall = "10921293373", Climb = "10921286911", Swim = "10921295495", SwimIdle = "10921297391" },
    WereWolf = { Idle1 = "10921330408", Idle2 = "10921333667", Walk = "10921342074", Run = "10921336997", Jump = "1083218792", Fall = "10921337907", Climb = "10921329322", Swim = "10921340419", SwimIdle = "10921341319" },
    Vampire = { Idle1 = "10921315373", Idle2 = "10921316709", Walk = "10921326949", Run = "10921320299", Jump = "10921322186", Fall = "10921321317", Climb = "10921314188", Swim = "10921324408", SwimIdle = "10921325443" },
    Astronaut = { Idle1 = "10921034824", Idle2 = "10921036806", Walk = "10921046031", Run = "10921039308", Jump = "10921042494", Fall = "10921040576", Climb = "10921032124", Swim = "10921044000", SwimIdle = "10921045006" },
    Malvada = { Idle1 = "118832222982049", Idle2 = "76049494037641", Walk = "92072849924640", Run = "72301599441680", Jump = "104325245285198", Fall = "121152442762481", Climb = "131326830509784", Swim = "99384245425157", SwimIdle = "113199415118199" },
    Walmart = { Idle1 = "18747067405", Idle2 = "18747063918", Walk = "18747074203", Run = "18747070484", Jump = "18747069148", Fall = "18747062535", Climb = "18747060903", Swim = "18747073181", SwimIdle = "18747071682" },
    Dancing = { Idle1 = "92849173543269", Idle2 = "132238900951109", Walk = "73718308412641", Run = "135515454877967", Jump = "78508480717326", Fall = "78147885297412", Climb = "129447497744818", Swim = "110657013921774", SwimIdle = "129183123083281" },
    RunwayGlamour = { Idle1 = "133806214992291", Idle2 = "94970088341563", Walk = "109168724482748", Run = "81024476153754", Jump = "116936326516985", Fall = "92294537340807", Climb = "119377220967554", Swim = "134591743181628", SwimIdle = "98854111361360" },
    Amazon = { Idle1 = "98281136301627", Idle2 = "138183121662404", Walk = "90478085024465", Run = "134824450619865", Jump = "121454505477205", Fall = "94788218468396", Climb = "121145883950231", Swim = "105962919001086", SwimIdle = "129126268464847" },
    Mage = { Idle1 = "10921144709", Idle2 = "10921145797", Walk = "10921152678", Run = "10921148209", Jump = "10921149743", Fall = "10921148939", Climb = "10921143404", Swim = "10921150788", SwimIdle = "10921151661" },
    Toy = { Idle1 = "10921301576", Idle2 = "10921302207", Walk = "10921312010", Run = "10921306285", Jump = "10921308158", Fall = "10921307241", Climb = "10921300839", Swim = "10921309319", SwimIdle = "10921310341" },
    Elder = { Idle1 = "10921101664", Idle2 = "10921102574", Walk = "10921111375", Run = "10921104374", Jump = "10921107367", Fall = "10921105765", Climb = "10921100400", Swim = "10921108971", SwimIdle = "10921110146" },
    AdidasAura = { Idle1 = "110211186840347", Idle2 = "114191137265065", Walk = "83842218823011", Run = "118320322718866", Jump = "109996626521204", Fall = "95603166884636", Climb = "97824616490448", Swim = "134530128383903", SwimIdle = "94922130551805" },
    Classic = { Idle1 = "10921230744", Idle2 = "10921232093", Walk = "10921244891", Run = "10921240218", Jump = "10921242013", Fall = "10921241244", Climb = "10921229866", Swim = "10921243048", SwimIdle = "10921244018" },
    Stylish = { Idle1 = "10921272275", Idle2 = "10921273958", Walk = "10921283326", Run = "10921276116", Jump = "10921279832", Fall = "10921278648", Climb = "10921271391", Swim = "10921281000", SwimIdle = "10921281964" },
    Sports = { Idle1 = "18537376492", Idle2 = "18537371272", Walk = "18537392113", Run = "18537384940", Jump = "18537380791", Fall = "18537367238", Climb = "18537363391", Swim = "18537389531", SwimIdle = "18537387180" },
    AdidasCommunity = { Idle1 = "122257458498464", Idle2 = "102357151005774", Walk = "122150855457006", Run = "82598234841035", Jump = "75290611992385", Fall = "98600215928904", Climb = "88763136693023", Swim = "133308483266208", SwimIdle = "109346520324160" },
    Robot = { Idle1 = "10921248039", Idle2 = "10921248831", Walk = "10921255446", Run = "10921250460", Jump = "10921252123", Fall = "10921251156", Climb = "10921247141", Swim = "10921253142", SwimIdle = "10921253767" },
    Glow = { Idle1 = "137764781910579", Idle2 = "96439737641086", Walk = "85809016093530", Run = "101925097435036", Jump = "74159004634379", Fall = "98070939608691", Climb = "108236155509584", Swim = "83003487432457", SwimIdle = "112946194103503" },
    Bold = { Idle1 = "16738333868", Idle2 = "16738334710", Walk = "16738340646", Run = "16738337225", Jump = "16738336650", Fall = "16738333171", Climb = "16738332169", Swim = "16738339158", SwimIdle = "16738339817" },
    Levitation = { Idle1 = "10921132962", Idle2 = "10921133721", Walk = "10921140719", Run = "10921135644", Jump = "10921137402", Fall = "10921136539", Climb = "10921132092", Swim = "10921138209", SwimIdle = "10921139478" }
}

local EMOTES = {
    {name = "Footwork Funk Brasileiro 2", id = "140219184038687"},
    {name = "Sentar", id = "86052354840008"},
    {name = "Passinho do Jamal", id = "83796130837213"},
    {name = "Rocker Festa", id = "97500383729078"},
    {name = "Kareen", id = "131703052643885"},
    {name = "Popular", id = "94292758607678"},
    {name = "Falso Morto MM2", id = "110770090804022"},
    {name = "Cagar", id = "121561448415799"},
    {name = "Dança Hype", id = "129200076968240"},
    {name = "Abraçar", id = "89634365084537"},
    {name = "Rebolada", id = "86256296565849"}
}

local function aplicarPack(pack)
    if not pack then return end
    local char = player.Character or player.CharacterAdded:Wait()
    local animate = char:WaitForChild("Animate")
    
    local function set(folder, id)
        if not folder then return end
        for _, v in pairs(folder:GetChildren()) do
            if v:IsA("Animation") then
                v.AnimationId = "rbxassetid://" .. id
            end
        end
    end
    
    set(animate:FindFirstChild("idle"), pack.Idle1)
    set(animate:FindFirstChild("walk"), pack.Walk)
    set(animate:FindFirstChild("run"), pack.Run)
    set(animate:FindFirstChild("jump"), pack.Jump)
    set(animate:FindFirstChild("fall"), pack.Fall)
    set(animate:FindFirstChild("climb"), pack.Climb)
    set(animate:FindFirstChild("swim"), pack.Swim)
    
    char:WaitForChild("Humanoid"):ChangeState(Enum.HumanoidStateType.Running)
end

local function playEmote(id, speed)
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    local animator = humanoid:FindFirstChild("Animator")
    if not animator then
        animator = Instance.new("Animator")
        animator.Parent = humanoid
    end
    
    if currentEmoteTrack then
        currentEmoteTrack:Stop()
        currentEmoteTrack = nil
    end
    
    local success = pcall(function()
        humanoid:PlayEmote(tostring(id))
    end)
    
    if success then
        return
    end
    
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://" .. tostring(id)
    local track = animator:LoadAnimation(anim)
    
    if track then
        track.Priority = Enum.AnimationPriority.Action4
        track.Looped = Settings.loopEmote
        track:Play()
        
        if speed and speed ~= 1 then
            track:AdjustSpeed(speed)
        end
        
        currentEmoteTrack = track
        
        if not Settings.loopEmote then
            local moveConnection
            moveConnection = humanoid:GetPropertyChangedSignal("MoveDirection"):Connect(function()
                if humanoid.MoveDirection.Magnitude > 0 then
                    track:Stop()
                    moveConnection:Disconnect()
                    if currentEmoteTrack == track then
                        currentEmoteTrack = nil
                    end
                end
            end)
        end
    else
        AddNotification("OZ HUB", "Emote nao encontrado", 3)
    end
end

local function stopEmote()
    if currentEmoteTrack then
        currentEmoteTrack:Stop()
        currentEmoteTrack = nil
    end
end

-- ============================================
--  MM2 FUNCTIONS
-- ============================================

local function encontrarPorPapel(papelAlvo)
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and getPapel(plr) == papelAlvo then
            return plr
        end
    end
    return nil
end

local function pararJogarAssassino()
    if Settings.killAssassinActive then
        Settings.killAssassinActive = false
        if Settings.killAssassinConnection then
            Settings.killAssassinConnection:Disconnect()
            Settings.killAssassinConnection = nil
        end
        LoopManager:Remove("KillAssassinFling")
        local meuHRP = GetHRP()
        if meuHRP then
            meuHRP.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            meuHRP.RotVelocity = Vector3.new(0, 0, 0)
            meuHRP.Velocity = Vector3.new(0, 0, 0)
            if Settings.killAssassinPos then
                meuHRP.CFrame = Settings.killAssassinPos
                Settings.killAssassinPos = nil
            end
        end
    end
end

local function pararJogarXerife()
    if Settings.killSheriffActive then
        Settings.killSheriffActive = false
        if Settings.killSheriffConnection then
            Settings.killSheriffConnection:Disconnect()
            Settings.killSheriffConnection = nil
        end
        LoopManager:Remove("KillSheriffFling")
        local meuHRP = GetHRP()
        if meuHRP then
            meuHRP.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            meuHRP.RotVelocity = Vector3.new(0, 0, 0)
            meuHRP.Velocity = Vector3.new(0, 0, 0)
            if Settings.killSheriffPos then
                meuHRP.CFrame = Settings.killSheriffPos
                Settings.killSheriffPos = nil
            end
        end
    end
end

local function monitorarAssassino()
    local assassino = encontrarPorPapel("Assassino")
    if assassino and assassino.Character then
        local humanoid = GetHumanoid(assassino.Character)
        if humanoid then
            if Settings.killAssassinConnection then
                Settings.killAssassinConnection:Disconnect()
            end
            Settings.killAssassinConnection = humanoid.Died:Connect(function()
                if Settings.killAssassinActive then
                    pararJogarAssassino()
                end
            end)
        end
    end
end

local function monitorarXerife()
    local xerife = encontrarPorPapel("Xerife")
    if xerife and xerife.Character then
        local humanoid = GetHumanoid(xerife.Character)
        if humanoid then
            if Settings.killSheriffConnection then
                Settings.killSheriffConnection:Disconnect()
            end
            Settings.killSheriffConnection = humanoid.Died:Connect(function()
                if Settings.killSheriffActive then
                    pararJogarXerife()
                end
            end)
        end
    end
end

-- ============================================
--  BOTÃO FLUTUANTE E UI PRINCIPAL
-- ============================================

local Window = WindUI:CreateWindow({
    Title = "OZ HUB",
    Folder = "OzHub",
    Size = UDim2.fromOffset(580, 460),
    Transparent = true,
    Theme = "Dark",
    Resizable = false,
    SideBarWidth = 200,
    HideSearchBar = true,
    ScrollBarEnabled = false,
    User = { Enabled = true, Anonymous = false }
})

pcall(function() Window:EditOpenButton({ Enabled = false }) end)

local floatingGui = Instance.new("ScreenGui")
floatingGui.Name = "OzHubFloatingButton"
floatingGui.IgnoreGuiInset = true
floatingGui.ResetOnSpawn = false
floatingGui.Parent = game.CoreGui

local floatingButton = Instance.new("ImageButton")
floatingButton.Size = UDim2.new(0, 45, 0, 45)
floatingButton.Position = UDim2.new(0, 70, 0, 70)
floatingButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
floatingButton.Image = "rbxassetid://87785485179706"
floatingButton.Name = "OzHubToggle"
floatingButton.AutoButtonColor = true
floatingButton.Parent = floatingGui

local floatingCorner = Instance.new("UICorner", floatingButton)
floatingCorner.CornerRadius = UDim.new(0, 10)

local floatingStroke = Instance.new("UIStroke")
floatingStroke.Thickness = 2
floatingStroke.Color = Color3.fromRGB(255, 100, 100)
floatingStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
floatingStroke.Parent = floatingButton

local floatingRgbHue = 0
local floatingRainbowConnection = RunService.RenderStepped:Connect(function()
    floatingRgbHue = (floatingRgbHue + 0.005) % 1
    floatingStroke.Color = Color3.fromHSV(floatingRgbHue, 1, 1)
end)

EventManager:Add("FloatingButtonRainbow", floatingRainbowConnection)

local dragging = false
local dragInput = nil
local dragStart = nil
local startPos = nil

floatingButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = floatingButton.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

floatingButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        floatingButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

floatingButton.MouseButton1Click:Connect(function()
    if Window.Visible then
        Window:Close()
    else
        Window:Open()
    end
end)

-- ============================================
--  CRIAÇÃO DAS ABAS
-- ============================================

local AimbotTab = Window:Tab({Title = "Aimbot", Icon = "crosshair"})

AimbotTab:Toggle({
    Title = "Ativar Aimbot",
    Default = false,
    Callback = function(Value)
        Settings.AimbotEnabled = Value
        if Value then
            LoopManager:Add("Aimbot", AimbotTask, 0.05)
        else
            LoopManager:Remove("Aimbot")
        end
        UpdateFOVCircle()
    end
})

AimbotTab:Dropdown({
    Title = "Target",
    Values = {"Head", "Torso"},
    Default = "Head",
    Callback = function(Value)
        Settings.SelectedAimPart = Value
    end
})

AimbotTab:Slider({
    Title = "Exibir Circulo do FOV",
    Step = 1,
    Value = {Min = 30, Max = 180, Default = 90},
    Callback = function(Value)
        Settings.AimbotFOV = Value
        UpdateFOVCircle()
    end
})

AimbotTab:Slider({
    Title = "Smoothing",
    Step = 1,
    Value = {Min = 0, Max = 10, Default = 5},
    Callback = function(Value)
        Settings.AimbotSmoothness = Value
    end
})

AimbotTab:Toggle({
    Title = "Ignore Walls",
    Default = false,
    Callback = function(Value)
        Settings.IgnoreWalls = Value
    end
})

local VisualsTab = Window:Tab({Title = "Visuais", Icon = "eye"})

ESPNormalToggle = VisualsTab:Toggle({
    Title = "Enabled ESP",
    Default = false,
    Callback = function(Value)
        Settings.ESPHighlightEnabled = Value
        if Value then
            iniciarESPHighlight()
            AddNotification("OZ HUB", "ESP ativado", 3)
            if ESPMM2Toggle then ESPMM2Toggle:Lock() end
        else
            if not Settings.ESPMM2HighlightEnabled then
                pararESPHighlight()
                AddNotification("OZ HUB", "ESP desativado", 3)
            end
            if ESPMM2Toggle then ESPMM2Toggle:Unlock() end
        end
    end
})

VisualsTab:Toggle({
    Title = "Hide @User (Clean Names)",
    Default = false,
    Callback = function(Value)
        Settings.HideNameEnabled = Value
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local humanoid = GetHumanoid(plr.Character)
                if humanoid then
                    humanoid.DisplayDistanceType = Value and Enum.HumanoidDisplayDistanceType.None or Enum.HumanoidDisplayDistanceType.Viewer
                end
            end
        end
        if Value then
            AddNotification("OZ HUB", "Nomes ocultos", 3)
        else
            AddNotification("OZ HUB", "Nomes visiveis", 3)
        end
    end
})

VisualsTab:Toggle({
    Title = "Info Panel",
    Default = false,
    Callback = function(Value)
        Settings.ShowFPSEnabled = Value
        if Value then
            CreateInfoPanel()
            AddNotification("OZ HUB", "Info Panel ativado", 3)
        else
            if InfoPanel then InfoPanel:Destroy() end
            InfoPanel = nil
            if InfoPanelConnection then InfoPanelConnection:Disconnect() end
            InfoPanelConnection = nil
            if InfoPanelRainbowConnection then InfoPanelRainbowConnection:Disconnect() end
            InfoPanelRainbowConnection = nil
            AddNotification("OZ HUB", "Info Panel desativado", 3)
        end
    end
})

VisualsTab:Section({Title = "Hub Cosmetics"})

VisualsTab:Toggle({
    Title = "Headless (Visual Only)",
    Default = false,
    Callback = function(Value)
        Settings.headlessActive = Value
        toggleHeadless(Value)
        if Value then
            AddNotification("OZ HUB", "Headless ativado", 3)
        else
            AddNotification("OZ HUB", "Headless desativado", 3)
        end
    end
})

VisualsTab:Toggle({
    Title = "Korblox (Visual Only)",
    Default = false,
    Callback = function(Value)
        Settings.korbloxActive = Value
        toggleKorblox(Value)
        if Value then
            AddNotification("OZ HUB", "Korblox ativado", 3)
        else
            AddNotification("OZ HUB", "Korblox desativado", 3)
        end
    end
})

local ExploitTab = Window:Tab({Title = "Exploit", Icon = "box"})

ExploitTab:Section({Title = "Movement & Tools"})

ExploitTab:Toggle({
    Title = "Fly",
    Default = false,
    Callback = function(Value)
        toggleFly(Value)
        if Value then
            AddNotification("OZ HUB", "Fly ativado", 3)
        else
            AddNotification("OZ HUB", "Fly desativado", 3)
        end
    end
})

ExploitTab:Slider({
    Title = "Fly Speed",
    Step = 1,
    Value = {Min = 10, Max = 300, Default = 50},
    Callback = function(Value)
        Settings.FlySpeed = Value
    end
})

ExploitTab:Toggle({
    Title = "Speed",
    Default = false,
    Callback = function(Value)
        Settings.SpeedToggleEnabled = Value
        if Value then
            aplicarSpeed()
            AddNotification("OZ HUB", "Speed ativado", 3)
        else
            reverterSpeed()
            AddNotification("OZ HUB", "Speed desativado", 3)
        end
    end
})

ExploitTab:Slider({
    Title = "Speed Walk",
    Step = 1,
    Value = {Min = 16, Max = 150, Default = 16},
    Callback = function(Value)
        Settings.SpeedValue = Value
        if Settings.SpeedToggleEnabled then
            aplicarSpeed()
        end
    end
})

ExploitTab:Toggle({
    Title = "FOV Cam",
    Default = false,
    Callback = function(Value)
        Settings.FOVCamToggleEnabled = Value
        if Value then
            aplicarFov()
            AddNotification("OZ HUB", "FOV Cam ativado", 3)
        else
            reverterFov()
            AddNotification("OZ HUB", "FOV Cam desativado", 3)
        end
    end
})

ExploitTab:Slider({
    Title = "FOV Cam (Zoom)",
    Step = 1,
    Value = {Min = 1, Max = 120, Default = 70},
    Callback = function(Value)
        Settings.FovCamValue = Value
        if Settings.FOVCamToggleEnabled then
            aplicarFov()
        end
    end
})

ExploitTab:Toggle({
    Title = "Infinite Jump",
    Default = false,
    Callback = function(Value)
        Settings.infJumpEnabled = Value
        if Value then
            AddNotification("OZ HUB", "Infinite Jump ativado", 3)
        else
            AddNotification("OZ HUB", "Infinite Jump desativado", 3)
        end
    end
})

NoClipToggle = ExploitTab:Toggle({
    Title = "NoClip",
    Default = false,
    Callback = function(Value)
        if Settings.AutoFarmEnabled then
            NoClipToggle:Set(Settings.NoClipEnabled)
            return
        end
        Settings.NoClipEnabled = Value
        if Value then
            aplicarNoClip()
            if AutoFarmToggleElement then
                AutoFarmToggleElement:Lock()
            end
            AddNotification("OZ HUB", "NoClip ativado", 3)
        else
            reverterNoClip()
            if AutoFarmToggleElement then
                AutoFarmToggleElement:Unlock()
            end
            AddNotification("OZ HUB", "NoClip desativado", 3)
        end
    end
})

ExploitTab:Section({Title = "Tools & Utilities"})

ExploitTab:Toggle({
    Title = "Spinbot",
    Default = false,
    Callback = function(Value)
        toggleSpinbot(Value)
        if Value then
            AddNotification("OZ HUB", "Spinbot ativado", 3)
        else
            AddNotification("OZ HUB", "Spinbot desativado", 3)
        end
    end
})

ExploitTab:Toggle({
    Title = "Lock Cam",
    Default = false,
    Callback = function(Value)
        Settings.LockCamEnabled = Value
        if Value then
            CreateShiftLockButton()
            CriarMira()
            LoopManager:Add("LockCam", LockCamTask, 0)
            AddNotification("OZ HUB", "Lock Cam ativado", 3)
        else
            if LockUI then
                LockUI:Destroy()
                LockUI = nil
            end
            if MiraUI then
                MiraUI:Destroy()
                MiraUI = nil
            end
            LoopManager:Remove("LockCam")
            local char = GetChar()
            if char then
                local humanoid = GetHumanoid(char)
                if humanoid then
                    humanoid.CameraOffset = Vector3.new(0, 0, 0)
                end
            end
            AddNotification("OZ HUB", "Lock Cam desativado", 3)
        end
    end
})

ExploitTab:Button({
    Title = "Click TP",
    Callback = function()
        if not player.Character then return end
        local tool = Instance.new("Tool")
        tool.Name = "TP Tool"
        tool.RequiresHandle = false
        tool.CanBeDropped = false
        tool.TextureId = "rbxassetid://13060727541"
        tool.Activated:Connect(function()
            local mouse = player:GetMouse()
            local hit = mouse.Hit
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local rayOrigin = hit.Position + Vector3.new(0, 5, 0)
                local rayDirection = Vector3.new(0, -10, 0)
                local raycastParams = RaycastParams.new()
                raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                raycastParams.FilterDescendantsInstances = {player.Character}
                local rayResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
                if rayResult and rayResult.Instance and rayResult.Instance.CanCollide then
                    player.Character.HumanoidRootPart.CFrame = CFrame.new(rayResult.Position + Vector3.new(0, 3, 0))
                elseif hit.Position.Y > 3 and hit.Position.Y < 1000 then
                    player.Character.HumanoidRootPart.CFrame = CFrame.new(hit.Position + Vector3.new(0, 3, 0))
                end
            end
        end)
        tool.Parent = player.Backpack
        AddNotification("OZ HUB", "Click TP adicionado ao inventario", 3)
    end
})

ExploitTab:Button({
    Title = "Reiniciar",
    Callback = function()
        local char = GetChar()
        if not char then
            AddNotification("OZ HUB", "Personagem nao encontrado", 3)
            return
        end
        local hrp = GetHRP(char)
        if not hrp then
            AddNotification("OZ HUB", "HumanoidRootPart nao encontrado", 3)
            return
        end
        local posicaoAtual = hrp.CFrame
        local humanoid = GetHumanoid(char)
        if humanoid then
            humanoid.Health = 0
        end
        local newChar = player.CharacterAdded:Wait()
        local newHRP = GetHRP(newChar)
        if newHRP then
            newHRP.CFrame = posicaoAtual
        end
        AddNotification("OZ HUB", "Personagem reiniciado", 3)
    end
})

-- ========== ONLINES TAB OTIMIZADA ==========
local OnlinesTab = Window:Tab({Title = "Onlines", Icon = "users"})

local infoSection = OnlinesTab:Section({ Title = "Informações do Jogador", Opened = true })
local infoParagraph = nil

local function updateInfoParagraph(plr)
    if infoParagraph then
        pcall(function() infoParagraph:Destroy() end)
    end
    if not plr then return end
    
    infoParagraph = infoSection:Paragraph({
        Title = plr.DisplayName .. " (" .. plr.Name .. ")",
        Desc = "ID: " .. plr.UserId,
        Image = getHeadshot(plr),
        ImageSize = 50,
    })
end

updateInfoParagraph(player)

local thumbnailCache = {}

local function getCachedHeadshot(plr)
    if thumbnailCache[plr] then
        return thumbnailCache[plr]
    end
    local thumb = getHeadshot(plr)
    thumbnailCache[plr] = thumb
    return thumb
end

local function atualizarListaJogadores()
    local lista = {}
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            table.insert(lista, {
                Title = plr.Name,
                Icon = getCachedHeadshot(plr),
                Player = plr,
            })
        end
    end
    return lista
end

local playerDropdown = OnlinesTab:Dropdown({
    Title = "Selecionar Jogador",
    Values = atualizarListaJogadores(),
    Default = "Ninguem",
    Callback = function(opt)
        if opt and opt.Player then
            currentSelectedPlayer = opt.Player
            updateInfoParagraph(currentSelectedPlayer)
        end
    end
})

OnlinesTab:Section({ Title = "Acoes" })

OnlinesTab:Button({
    Title = "Copy ID",
    Callback = function()
        if not currentSelectedPlayer then
            AddNotification("OZ HUB", "Selecione um jogador primeiro", 3)
            return
        end
        if setclipboard then
            setclipboard(tostring(currentSelectedPlayer.UserId))
            AddNotification("OZ HUB", "ID copiado: " .. currentSelectedPlayer.UserId, 3)
        end
    end
})

OnlinesTab:Button({
    Title = "Spectate",
    Callback = function()
        if not currentSelectedPlayer then
            AddNotification("OZ HUB", "Selecione um jogador primeiro", 3)
            return
        end
        stopAll()
        local char = currentSelectedPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            camera.CameraSubject = char.Humanoid
            camera.CameraType = Enum.CameraType.Custom
            Settings.spectateActive = true
            AddNotification("OZ HUB", "Spectate em: " .. currentSelectedPlayer.Name, 3)
        else
            AddNotification("OZ HUB", "Jogador sem personagem", 3)
        end
    end
})

OnlinesTab:Button({
    Title = "Teleport",
    Callback = function()
        if not currentSelectedPlayer then
            AddNotification("OZ HUB", "Selecione um jogador primeiro", 3)
            return
        end
        stopAll()
        local targetChar = currentSelectedPlayer.Character
        local myChar = GetChar()
        if targetChar and targetChar:FindFirstChild("HumanoidRootPart") and myChar and myChar:FindFirstChild("HumanoidRootPart") then
            myChar.HumanoidRootPart.CFrame = targetChar.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
            AddNotification("OZ HUB", "Teleportado para: " .. currentSelectedPlayer.Name, 3)
        else
            AddNotification("OZ HUB", "Jogador sem personagem", 3)
        end
    end
})

OnlinesTab:Button({
    Title = "Mochila",
    Callback = function()
        if not currentSelectedPlayer then
            AddNotification("OZ HUB", "Selecione um jogador primeiro", 3)
            return
        end
        if Settings.backpackActive then
            stopAll()
            AddNotification("OZ HUB", "Mochila desativado", 3)
            return
        end
        stopAll()
        Settings.backpackActive = true
        local targetPlayer = currentSelectedPlayer
        LoopManager:Add("Backpack", function()
            if not Settings.backpackActive or not targetPlayer or not targetPlayer.Character then
                LoopManager:Remove("Backpack")
                return
            end
            local myChar = GetChar()
            if myChar and myChar:FindFirstChild("HumanoidRootPart") then
                local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                if targetRoot then
                    myChar.HumanoidRootPart.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 1.4) * CFrame.Angles(0, math.rad(180), 0)
                    if myChar.Humanoid and myChar.Humanoid:GetState() ~= Enum.HumanoidStateType.Seated then
                        myChar.Humanoid.Sit = true
                    end
                end
            end
        end, 0)
        AddNotification("OZ HUB", "Mochila ativado em: " .. currentSelectedPlayer.Name, 3)
    end
})

OnlinesTab:Button({
    Title = "Headsit",
    Callback = function()
        if not currentSelectedPlayer then
            AddNotification("OZ HUB", "Selecione um jogador primeiro", 3)
            return
        end
        if Settings.headsitActive then
            stopAll()
            AddNotification("OZ HUB", "Headsit desativado", 3)
            return
        end
        stopAll()
        Settings.headsitActive = true
        local targetPlayer = currentSelectedPlayer
        LoopManager:Add("Headsit", function()
            if not Settings.headsitActive or not targetPlayer or not targetPlayer.Character then
                LoopManager:Remove("Headsit")
                return
            end
            local myChar = GetChar()
            if myChar and myChar:FindFirstChild("HumanoidRootPart") then
                local targetHead = targetPlayer.Character:FindFirstChild("Head")
                local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
                if targetHead and targetRoot then
                    local headPos = targetHead.Position + Vector3.new(0, 2.5, 0)
                    local lookDirection = targetRoot.CFrame.LookVector
                    lookDirection = Vector3.new(lookDirection.X, 0, lookDirection.Z).Unit
                    myChar.HumanoidRootPart.CFrame = CFrame.lookAt(headPos, headPos + lookDirection)
                    if myChar.Humanoid and myChar.Humanoid:GetState() ~= Enum.HumanoidStateType.Seated then
                        myChar.Humanoid.Sit = true
                    end
                end
            end
        end, 0)
        AddNotification("OZ HUB", "Headsit ativado em: " .. currentSelectedPlayer.Name, 3)
    end
})

--[[
OnlinesTab:Button({
    Title = "Fling OP",
    Callback = function()
        if not currentSelectedPlayer then
            AddNotification("OZ HUB", "Selecione um jogador primeiro", 3)
            return
        end
        if Settings.flingOPActive then
            stopAll()
            AddNotification("OZ HUB", "Fling OP desativado", 3)
            return
        end
        stopAll()
        
        local targetChar = currentSelectedPlayer.Character
        local myChar = GetChar()
        if not targetChar or not myChar then
            AddNotification("OZ HUB", "Personagem nao encontrado", 3)
            return
        end
        
        local targetHRP = GetHRP(targetChar)
        local myHRP = GetHRP(myChar)
        if not targetHRP or not myHRP then
            AddNotification("OZ HUB", "HumanoidRootPart nao encontrado", 3)
            return
        end
        
        posicaoOriginal = myHRP.CFrame
        Settings.flingOPActive = true
        
        myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, 0)
        myHRP.RotVelocity = Vector3.new(9999, 9999, 9999)
        
        local angulo = 0
        local targetCache = currentSelectedPlayer
        
        LoopManager:Add("FlingOP", function()
            if not Settings.flingOPActive or not targetCache or not targetCache.Character then
                LoopManager:Remove("FlingOP")
                return
            end
            
            local myHRP = GetHRP()
            local targetHRP = GetHRP(targetCache.Character)
            
            if myHRP and targetHRP then
                angulo = angulo + 0.3
                myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, 0) * CFrame.Angles(math.sin(angulo) * 3, 3, 3)
                myHRP.RotVelocity = Vector3.new(9999, 9999, 9999)
                myHRP.Velocity = (targetHRP.Position - myHRP.Position).Unit * 9999
            else
                stopAll()
            end
        end        
        AddNotification("OZ HUB", "Fling OP ativado em: " .. currentSelectedPlayer.Name, 3)
    end
})]]--

OnlinesTab:Button({
    Title = "Stop All",
    Callback = function()
        if not (Settings.backpackActive or Settings.headsitActive or Settings.spectateActive or Settings.flingOPActive) then
            AddNotification("OZ HUB", "Nenhuma acao ativa", 3)
            return
        end
        stopAll()
        AddNotification("OZ HUB", "Todas as acoes paradas", 3)
    end
})

local function atualizarDropdown()
    local novaLista = {}
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            table.insert(novaLista, {
                Title = plr.Name,
                Icon = getCachedHeadshot(plr),
                Player = plr,
            })
        end
    end
    playerDropdown:Refresh(novaLista)
end

Players.PlayerAdded:Connect(atualizarDropdown)
Players.PlayerRemoving:Connect(function(plr)
    if currentSelectedPlayer == plr then
        currentSelectedPlayer = nil
        updateInfoParagraph(player)
    end
    thumbnailCache[plr] = nil
    atualizarDropdown()
end)

local MM2Tab = Window:Tab({Title = "Murder Mystery 2", Icon = "axe"})

MM2Tab:Section({Title = "Visuais"})

ESPMM2Toggle = MM2Tab:Toggle({
    Title = "ESP de Jogadores",
    Default = false,
    Callback = function(Value)
        Settings.ESPMM2HighlightEnabled = Value
        if Value then
            iniciarESPHighlight()
            AddNotification("OZ HUB", "ESP MM2 ativado", 3)
            if ESPNormalToggle then ESPNormalToggle:Lock() end
        else
            if not Settings.ESPHighlightEnabled then
                pararESPHighlight()
                AddNotification("OZ HUB", "ESP MM2 desativado", 3)
            end
            if ESPNormalToggle then ESPNormalToggle:Unlock() end
        end
    end
})

MM2Tab:Section({Title = "Combat"})

MM2Tab:Button({
    Title = "Puxar Xerife",
    Callback = function()
        local xerife = encontrarPorPapel("Xerife")
        if xerife and xerife.Character then
            local targetHRP = GetHRP(xerife.Character)
            local myHRP = GetHRP()
            if targetHRP and myHRP then
                targetHRP.CFrame = myHRP.CFrame
                AddNotification("OZ HUB", "Xerife puxado", 3)
            end
        else
            AddNotification("OZ HUB", "Xerife nao encontrado", 3)
        end
    end
})

MM2Tab:Button({
    Title = "Puxar Todos",
    Callback = function()
        local myHRP = GetHRP()
        if not myHRP then return end
        local myPos = myHRP.Position
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local targetHRP = GetHRP(plr.Character)
                if targetHRP then
                    local distancia = (targetHRP.Position - myPos).Magnitude
                    if distancia <= 250 then
                        targetHRP.CFrame = myHRP.CFrame
                        task.wait(0.1)
                    end
                end
            end
        end
        AddNotification("OZ HUB", "Todos puxados", 3)
    end
})

--[[MM2Tab:Button({
    Title = "Jogar Assassino Longe",
    Callback = function()
        if Settings.killAssassinActive then
            pararJogarAssassino()
            return
        end
        local assassino = encontrarPorPapel("Assassino")
        if not assassino then
            AddNotification("OZ HUB", "Nenhum assassino encontrado", 3)
            return
        end
        local targetChar = assassino.Character
        local myChar = GetChar()
        if not targetChar or not myChar then
            AddNotification("OZ HUB", "Personagem nao encontrado", 3)
            return
        end
        local targetHRP = GetHRP(targetChar)
        local myHRP = GetHRP(myChar)
        if not targetHRP or not myHRP then
            AddNotification("OZ HUB", "HumanoidRootPart nao encontrado", 3)
            return
        end
        Settings.killAssassinActive = true
        Settings.killAssassinPos = myHRP.CFrame
        
        myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, 0)
        myHRP.RotVelocity = Vector3.new(9999, 9999, 9999)
        
        local angulo = 0
        local targetCache = assassino
        
        LoopManager:Add("KillAssassinFling", function()
            if not Settings.killAssassinActive or not targetCache or not targetCache.Character then
                LoopManager:Remove("KillAssassinFling")
                return
            end
            
            local myHRP = GetHRP()
            local targetHRP = GetHRP(targetCache.Character)
            
            if myHRP and targetHRP then
                angulo = angulo + 0.3
                myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, 0) * CFrame.Angles(math.sin(angulo) * 3, 3, 3)
                myHRP.RotVelocity = Vector3.new(9999, 9999, 9999)
                myHRP.Velocity = (targetHRP.Position - myHRP.Position).Unit * 9999
            else
                pararJogarAssassino()
            end
        end        
        AddNotification("OZ HUB", "Jogando Assassino longe", 3)
    end
})

MM2Tab:Button({
    Title = "Jogar Xerife Longe",
    Callback = function()
        if Settings.killSheriffActive then
            pararJogarXerife()
            return
        end
        local xerife = encontrarPorPapel("Xerife")
        if not xerife then
            AddNotification("OZ HUB", "Nenhum xerife encontrado", 3)
            return
        end
        local targetChar = xerife.Character
        local myChar = GetChar()
        if not targetChar or not myChar then
            AddNotification("OZ HUB", "Personagem nao encontrado", 3)
            return
        end
        local targetHRP = GetHRP(targetChar)
        local myHRP = GetHRP(myChar)
        if not targetHRP or not myHRP then
            AddNotification("OZ HUB", "HumanoidRootPart nao encontrado", 3)
            return
        end
        Settings.killSheriffActive = true
        Settings.killSheriffPos = myHRP.CFrame
        
        myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, 0)
        myHRP.RotVelocity = Vector3.new(9999, 9999, 9999)
        
        local angulo = 0
        local targetCache = xerife
        
        LoopManager:Add("KillSheriffFling", function()
            if not Settings.killSheriffActive or not targetCache or not targetCache.Character then
                LoopManager:Remove("KillSheriffFling")
                return
            end
            
            local myHRP = GetHRP()
            local targetHRP = GetHRP(targetCache.Character)
            
            if myHRP and targetHRP then
                angulo = angulo + 0.3
                myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, 0) * CFrame.Angles(math.sin(angulo) * 3, 3, 3)
                myHRP.RotVelocity = Vector3.new(9999, 9999, 9999)
                myHRP.Velocity = (targetHRP.Position - myHRP.Position).Unit * 9999
            else
                pararJogarXerife()
            end
        end
        AddNotification("OZ HUB", "Jogando Xerife longe", 3)
    end
})]]--

MM2Tab:Section({Title = "Utilidades"})

AutoFarmToggleElement = MM2Tab:Toggle({
    Title = "Auto Farm Moedas",
    Default = false,
    Callback = function(Value)
        if Settings.NoClipEnabled and Value then
            AddNotification("OZ HUB", "Desative o NoClip primeiro", 3)
            AutoFarmToggleElement:Set(false)
            return
        end
        AutoFarmToggle(Value)
    end
})

local GraphicsTab = Window:Tab({Title = "Graphics", Icon = "sparkle"})

GraphicsTab:Section({Title = "Time Control"})

GraphicsTab:Slider({
    Title = "Clock Time",
    Step = 1,
    Value = {Min = 0, Max = 24, Default = 12},
    Callback = function(Value)
        Lighting.ClockTime = Value
    end
})

GraphicsTab:Button({
    Title = "Morning (08:00)",
    Callback = function()
        Lighting.ClockTime = 8
    end
})

GraphicsTab:Button({
    Title = "Noon (14:00)",
    Callback = function()
        Lighting.ClockTime = 14
    end
})

GraphicsTab:Button({
    Title = "Night (00:00)",
    Callback = function()
        Lighting.ClockTime = 0
    end
})

GraphicsTab:Section({Title = "Performance"})

GraphicsTab:Toggle({
    Title = "Modo Performance",
    Default = false,
    Callback = function(Value)
        Settings.PerformanceMode = Value
        if Value then
            Settings.OriginalBrightness = Lighting.Brightness
            Settings.OriginalGlobalShadows = Lighting.GlobalShadows
            Lighting.Brightness = 1.5
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 500
            Lighting.OutdoorAmbient = Color3.fromRGB(150, 150, 150)
            workspace.Terrain.WaterWaveSize = 0
            workspace.Terrain.WaterReflectance = 0
            workspace.Terrain.WaterTransparency = 0
            setfpscap(240)
            AddNotification("OZ HUB", "Modo Performance ativado", 3)
        else
            if Settings.OriginalBrightness then Lighting.Brightness = Settings.OriginalBrightness end
            if Settings.OriginalGlobalShadows ~= nil then Lighting.GlobalShadows = Settings.OriginalGlobalShadows end
            Lighting.FogEnd = 100000
            Lighting.OutdoorAmbient = Color3.fromRGB(127, 127, 127)
            setfpscap(60)
            AddNotification("OZ HUB", "Modo Performance desativado", 3)
        end
    end
})

local AnimationsTab = Window:Tab({Title = "Animations", Icon = "layers"})

AnimationsTab:Section({Title = "Emote Player"})

AnimationsTab:Toggle({
    Title = "Loop Animation",
    Default = false,
    Callback = function(Value)
        Settings.loopEmote = Value
        if currentEmoteTrack then
            currentEmoteTrack.Looped = Value
        end
    end
})

AnimationsTab:Dropdown({
    Title = "Emotes",
    Values = {
        "Footwork Funk Brasileiro 2", "Sentar", "Passinho do Jamal", "Rocker Festa", "Kareen",
        "Popular", "Falso Morto MM2", "Cagar", "Dança Hype", "Abraçar", "Rebolada"
    },
    Default = "Nenhum",
    Callback = function(Value)
        for _, emote in pairs(EMOTES) do
            if emote.name == Value then
                playEmote(emote.id, Settings.emoteSpeed)
                break
            end
        end
    end
})

AnimationsTab:Button({
    Title = "Stop Animation",
    Callback = function()
        stopEmote()
    end
})

AnimationsTab:Slider({
    Title = "Emote Speed",
    Step = 1,
    Value = {Min = 1, Max = 10, Default = 1},
    Callback = function(Value)
        Settings.emoteSpeed = Value
        if currentEmoteTrack then
            currentEmoteTrack:AdjustSpeed(Value)
        end
    end
})

AnimationsTab:Section({Title = "Full Packs"})

AnimationsTab:Dropdown({
    Title = "Animation Packs",
    Values = {
        "Pirate", "Cartoony", "Hero", "Knight", "SuperHero", "WereWolf", "Vampire", "Astronaut",
        "Malvada", "Walmart", "Dancing", "RunwayGlamour", "Amazon", "Mage", "Toy", "Elder",
        "AdidasAura", "Classic", "Stylish", "Sports", "AdidasCommunity", "Robot", "Glow", "Bold", "Levitation"
    },
    Default = "Nenhum",
    Callback = function(Value)
        if Value == "Nenhum" then
            selectedAnimationPack = nil
        else
            selectedAnimationPack = Value
            aplicarPack(ANIMATION_PACKS[Value])
        end
    end
})

local TeleportTab = Window:Tab({Title = "Teleport", Icon = "map-pin"})

TeleportTab:Section({Title = "Principais"})

TeleportTab:Button({
    Title = "Save Pos",
    Callback = function()
        local char = GetChar()
        if not char then
            AddNotification("OZ HUB", "Personagem nao encontrado", 3)
            return
        end
        local hrp = GetHRP(char)
        if not hrp then
            AddNotification("OZ HUB", "HumanoidRootPart nao encontrado", 3)
            return
        end
        savedPosition = hrp.CFrame
        lastPositionTP = hrp.CFrame
        AddNotification("OZ HUB", "Posicao salva", 3)
    end
})

TeleportTab:Button({
    Title = "Load Pos",
    Callback = function()
        if not savedPosition then
            AddNotification("OZ HUB", "Nenhuma posicao salva", 3)
            return
        end
        local char = GetChar()
        if not char then
            AddNotification("OZ HUB", "Personagem nao encontrado", 3)
            return
        end
        local hrp = GetHRP(char)
        if not hrp then
            AddNotification("OZ HUB", "HumanoidRootPart nao encontrado", 3)
            return
        end
        lastPositionTP = hrp.CFrame
        hrp.CFrame = savedPosition
        AddNotification("OZ HUB", "Teleportado", 3)
    end
})

TeleportTab:Button({
    Title = "Return (Last Pos)",
    Callback = function()
        if not lastPositionTP then
            AddNotification("OZ HUB", "Nenhuma posicao anterior", 3)
            return
        end
        local char = GetChar()
        if not char then
            AddNotification("OZ HUB", "Personagem nao encontrado", 3)
            return
        end
        local hrp = GetHRP(char)
        if not hrp then
            AddNotification("OZ HUB", "HumanoidRootPart nao encontrado", 3)
            return
        end
        hrp.CFrame = lastPositionTP
        AddNotification("OZ HUB", "Retornado", 3)
    end
})

-- ============================================
--  EVENTOS E LIMPEZA
-- ============================================

jumpConnection = UserInputService.JumpRequest:Connect(function()
    if Settings.infJumpEnabled and not UserInputService:GetFocusedTextBox() then
        local humanoid = GetChar() and GetChar():FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:ChangeState("Jumping")
        end
    end
end)

EventManager:Add("CharacterAdded", player.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    
    if Settings.flying then startFly() end
    if Settings.headlessActive or Settings.korbloxActive then
        toggleHeadless(Settings.headlessActive)
        toggleKorblox(Settings.korbloxActive)
    end
    if Settings.NoClipEnabled then
	    aplicarNoClip()
	end
    if selectedAnimationPack then aplicarPack(ANIMATION_PACKS[selectedAnimationPack]) end
    if Settings.SpeedToggleEnabled then aplicarSpeed() end
    if Settings.FOVCamToggleEnabled then aplicarFov() end
end))

EventManager:Add("CharacterRemoving", player.CharacterRemoving:Connect(function()
    if bodyVelocity then bodyVelocity:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
    bodyVelocity, bodyGyro = nil, nil
end))

Players.PlayerAdded:Connect(updatePlayerCache)
Players.PlayerRemoving:Connect(updatePlayerCache)
updatePlayerCache()

EventManager:Add("PlayerAdded", Players.PlayerAdded:Connect(monitorarAssassino))
EventManager:Add("PlayerAdded2", Players.PlayerAdded:Connect(monitorarXerife))

for _, plr in pairs(Players:GetPlayers()) do
    EventManager:Add("CharAdded_" .. plr.Name, plr.CharacterAdded:Connect(monitorarAssassino))
    EventManager:Add("CharAdded2_" .. plr.Name, plr.CharacterAdded:Connect(monitorarXerife))
end

EventManager:Add("PlayerRemovingMM2", Players.PlayerRemoving:Connect(function(p)
    if Settings.killAssassinActive and p == encontrarPorPapel("Assassino") then
        pararJogarAssassino()
    end
    if Settings.killSheriffActive and p == encontrarPorPapel("Xerife") then
        pararJogarXerife()
    end
end))

task.spawn(function()
    while true do
        task.wait(300)
        for i, obj in pairs(coinCache) do
            if not obj or not obj.Parent then
                coinCache[i] = nil
            end
        end
        local now = tick()
        for target, data in pairs(RaycastCache) do
            if now - data.time > 30 then
                RaycastCache[target] = nil
            end
        end
        for target, data in pairs(lastPositions) do
            if now - data.time > 30 then
                lastPositions[target] = nil
            end
        end
        pcall(collectgarbage, "collect")
    end
end)

Window:OnDestroy(function()
    if LoopManager.conn then
        LoopManager.conn:Disconnect()
        LoopManager.conn = nil
    end
    LoopManager.tasks = {}
    
    if Settings.AimbotEnabled then
        LoopManager:Remove("Aimbot")
        Settings.AimbotEnabled = false
    end
    
    if Settings.ESPHighlightEnabled or Settings.ESPMM2HighlightEnabled then
        pararESPHighlight()
        Settings.ESPHighlightEnabled = false
        Settings.ESPMM2HighlightEnabled = false
    end
    
    if Settings.SpeedToggleEnabled then
        reverterSpeed()
        Settings.SpeedToggleEnabled = false
    end
    
    if Settings.FOVCamToggleEnabled then
        reverterFov()
        Settings.FOVCamToggleEnabled = false
    end
    
    if Settings.flying then
        toggleFly(false)
        Settings.flying = false
    end
    
	if Settings.NoClipEnabled then
	    reverterNoClip()
	    Settings.NoClipEnabled = false
	end
    
    if Settings.AutoFarmEnabled then
        AutoFarmToggle(false)
        Settings.AutoFarmEnabled = false
    end
    
    if Settings.SpinbotEnabled then
        toggleSpinbot(false)
        Settings.SpinbotEnabled = false
    end
    
    Settings.infJumpEnabled = false
    if jumpConnection then
        jumpConnection:Disconnect()
        jumpConnection = nil
    end
    
    if InfoPanelRainbowConnection then
	    InfoPanelRainbowConnection:Disconnect()
	    InfoPanelRainbowConnection = nil
	end

    if Settings.LockCamEnabled then
        Settings.LockCamEnabled = false
        if LockUI then LockUI:Destroy() end
        if MiraUI then MiraUI:Destroy() end
        LoopManager:Remove("LockCam")
        local char = GetChar()
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.CameraOffset = Vector3.new(0, 0, 0)
        end
    end
    
    if Settings.headlessActive then
        toggleHeadless(false)
        Settings.headlessActive = false
    end
    
    if Settings.korbloxActive then
        toggleKorblox(false)
        Settings.korbloxActive = false
    end
    
    if Settings.HideNameEnabled then
        Settings.HideNameEnabled = false
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local humanoid = GetHumanoid(plr.Character)
                if humanoid then
                    humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.Viewer
                end
            end
        end
    end
    
    stopAll()
    
    if Settings.killAssassinActive then
        pararJogarAssassino()
    end
    if Settings.killSheriffActive then
        pararJogarXerife()
    end
    
    if camera then
        camera.FieldOfView = 70
    end
    
    if Settings.PerformanceMode then
        if Settings.OriginalBrightness then Lighting.Brightness = Settings.OriginalBrightness end
        if Settings.OriginalGlobalShadows ~= nil then Lighting.GlobalShadows = Settings.OriginalGlobalShadows end
        Lighting.FogEnd = 100000
        Lighting.OutdoorAmbient = Color3.fromRGB(127, 127, 127)
        Settings.PerformanceMode = false
    end
    
    if FOVCircle then FOVCircle:Destroy() end
    if floatingGui then floatingGui:Destroy() end
    
    coinCache = {}
    playerCache = {}
    lastPositions = {}
    RaycastCache = {}
    ESPHighlights = {}
    
    for _, conn in pairs(EventManager.events) do
        pcall(function() conn:Disconnect() end)
    end
    EventManager.events = {}
    
    pcall(collectgarbage, "collect")
    
    AddNotification("OZ HUB", "Hub descarregado", 3)
end)
