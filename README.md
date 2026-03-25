local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
                            
local Window = WindUI:CreateWindow({
    Title = "SCARLET HUB",
    Author = "By Oliveira",
    Folder = "ScarletHub",
    Size = UDim2.fromOffset(600, 500),
    MinSize = Vector2.new(600, 500),
    MaxSize = Vector2.new(600, 500),
    Transparent = true,
    Theme = "Dark",
    Resizable = true,
    SideBarWidth = 200,
    HideSearchBar = true,
    ScrollBarEnabled = false,
    OpenButton = {
        Title = "S",
        Enabled = true,
        OnlyMobile = true,
        Draggable = true,
        CornerRadius = UDim.new(1, 0),
    },
    User = {
        Enabled = true,
        Anonymous = false
    }
})

-- ================================ FUNÇÕES GLOBAIS ================================

local function Notify(message, type)
    WindUI:Notify({
        Title = "SCARLET HUB",
        Content = message,
        Duration = 3,
        Icon = "info"
    })
    print("[" .. type:upper() .. "] " .. message)
end

-- ================================ SERVIÇOS ================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ================================ FUNÇÕES AUXILIARES ================================

local function GetChar()
    return player.Character
end

local function GetHRP(char)
    char = char or GetChar()
    return char and char:FindFirstChild("HumanoidRootPart")
end

local function GetHumanoid(char)
    char = char or GetChar()
    return char and char:FindFirstChild("Humanoid")
end

-- ================================ GERENCIADOR DE EVENTOS ================================

local EventManager = {
    events = {}
}

function EventManager:Add(name, connection)
    if self.events[name] then
        self.events[name]:Disconnect()
    end
    self.events[name] = connection
end

-- ================================ GERENCIADOR DE LOOPS ================================

local LoopManager = {
    frameTasks = {},
    throttledTasks = {},
    frameConn = nil,
    throttledConn = nil
}

function LoopManager:AddFrame(name, callback)
    if self.frameTasks[name] then return end
    self.frameTasks[name] = callback
    if not self.frameConn then
        self.frameConn = RunService.RenderStepped:Connect(function()
            for _, func in pairs(self.frameTasks) do
                pcall(func)
            end
        end)
    end
end

function LoopManager:AddThrottled(name, callback, interval)
    interval = interval or 0.3
    self.throttledTasks[name] = {func = callback, last = 0, interval = interval}
    if not self.throttledConn then
        self.throttledConn = RunService.Heartbeat:Connect(function()
            local now = tick()
            for _, task in pairs(self.throttledTasks) do
                if now - task.last >= task.interval then
                    task.last = now
                    pcall(task.func)
                end
            end
        end)
    end
end

function LoopManager:RemoveFrame(name)
    self.frameTasks[name] = nil
    if not next(self.frameTasks) and self.frameConn then
        self.frameConn:Disconnect()
        self.frameConn = nil
    end
end

function LoopManager:RemoveThrottled(name)
    self.throttledTasks[name] = nil
    if not next(self.throttledTasks) and self.throttledConn then
        self.throttledConn:Disconnect()
        self.throttledConn = nil
    end
end

function LoopManager:Add(name, callback)
    self:AddFrame(name, callback)
end

function LoopManager:Remove(name)
    self:RemoveFrame(name)
end

-- ================================ CACHE DE MOEDAS (OTIMIZADO) ================================

local coinCache = {}
local coinCacheDirty = true
local coinUpdateTimer = 0

local function atualizarCacheMoedas()
    if not coinCacheDirty then return end
    if tick() - coinUpdateTimer < 1 then return end
    coinUpdateTimer = tick()
    
    coinCache = {}

    for _, obj in pairs(workspace:GetChildren()) do
        if obj:IsA("Folder") or obj:IsA("Model") then
            for _, part in pairs(obj:GetDescendants()) do
                if part:IsA("BasePart") and part.Transparency < 0.8 then
                    local nome = part.Name:lower()
                    if nome:find("coin") or nome:find("gold") or nome:find("money") then
                        table.insert(coinCache, part)
                    end
                end
            end
        elseif obj:IsA("BasePart") then
            local nome = obj.Name:lower()
            if nome:find("coin") or nome:find("gold") or nome:find("money") then
                if obj.Transparency < 0.8 then
                    table.insert(coinCache, obj)
                end
            end
        end
    end
    coinCacheDirty = false
end

workspace.DescendantAdded:Connect(function()
    coinCacheDirty = true
end)

workspace.DescendantRemoving:Connect(function()
    coinCacheDirty = true
end)

-- ================================ CACHE DE PLAYERS ================================

local playerCache = {}
local playerCacheDirty = true

local function atualizarPlayerCache()
    if not playerCacheDirty then return end
    playerCache = {}
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            table.insert(playerCache, plr)
        end
    end
    playerCacheDirty = false
end

Players.PlayerAdded:Connect(function()
    playerCacheDirty = true
end)

Players.PlayerRemoving:Connect(function()
    playerCacheDirty = true
end)

-- ================================ CACHE DE PAPEL (OTIMIZADO) ================================

local papelCache = {}
local papelCacheDirty = true

local function getPapel(jogador)
    if not papelCacheDirty and papelCache[jogador] then
        return papelCache[jogador]
    end
    
    local char = jogador.Character
    if not char then return "Inocente" end
    
    local papel = "Inocente"
    if char:FindFirstChild("Knife") or char:FindFirstChild("FadeKnife") then
        papel = "Assassino"
    elseif char:FindFirstChild("Gun") or char:FindFirstChild("Pistol") or char:FindFirstChild("Revolver") then
        papel = "Xerife"
    elseif jogador.Backpack then
        if jogador.Backpack:FindFirstChild("Knife") or jogador.Backpack:FindFirstChild("FadeKnife") then
            papel = "Assassino"
        elseif jogador.Backpack:FindFirstChild("Gun") or jogador.Backpack:FindFirstChild("Pistol") or jogador.Backpack:FindFirstChild("Revolver") then
            papel = "Xerife"
        end
    end
    
    papelCache[jogador] = papel
    return papel
end

Players.PlayerAdded:Connect(function(plr)
    papelCache[plr] = nil
    plr.CharacterAdded:Connect(function()
        papelCache[plr] = nil
    end)
end)

Players.PlayerRemoving:Connect(function(plr)
    papelCache[plr] = nil
end)

-- ================================ ANIMAÇÕES ================================

local ANIMATION_PACKS = {
    Sneaky = { Idle1 = "1132473842", Idle2 = "1132477671", Walk = "1132510133", Run = "1132494274", Jump = "1132489853", Fall = "1132461372", Climb = "1132469004", Swim = "1132513516", SwimIdle = "1132516639" },
    Patrol = { Idle1 = "1149612882", Idle2 = "1150842221", Walk = "1151231493", Run = "1150967949", Jump = "1148811837", Fall = "1148863382", Climb = "1148811837", Swim = "1151204998", SwimIdle = "1151220343" },
    Ghost = { Idle1 = "616006778", Idle2 = "616008087", Walk = "616013216", Run = "616010382", Jump = "616008936", Fall = "616005863", Climb = "616003713", Swim = "616011509", SwimIdle = "616012453" },
    AdidasSports = { Idle1 = "18537371272", Idle2 = "18537376492", Walk = "18537392113", Run = "18537384940", Jump = "18537380791", Fall = "18537367238", Climb = "18537363391", Swim = "18537397079", SwimIdle = "18537402320" },
    AdidasCommunity = { Idle1 = "18537367238", Idle2 = "18537371272", Walk = "18537392113", Run = "18537384940", Jump = "18537380791", Fall = "18537375446", Climb = "18537369599", Swim = "18537397863", SwimIdle = "18537402207" },
    Levitation = { Idle1 = "616006778", Idle2 = "616008087", Walk = "616013216", Run = "616010382", Jump = "616008936", Fall = "616005863", Climb = "616003713", Swim = "616011509", SwimIdle = "616012453" },
    Werewolf = { Idle1 = "1083195517", Idle2 = "1083214717", Walk = "1083178339", Run = "1083216690", Jump = "1083218792", Fall = "1083189019", Climb = "1083182000", Swim = "1083222527", SwimIdle = "1083225406" },
    Vampire = { Idle1 = "1083445855", Idle2 = "1083450166", Walk = "1083473930", Run = "1083462077", Jump = "1083455352", Fall = "1083443587", Climb = "1083439238", Swim = "1083477583", SwimIdle = "1083480623" },
    Pirate = { Idle1 = "750781874", Idle2 = "750782770", Walk = "750785693", Run = "750783738", Jump = "750782230", Fall = "750780242", Climb = "750779899", Swim = "750784579", SwimIdle = "750785176" },
    Zombie = { Idle1 = "616158929", Idle2 = "616160636", Walk = "616168032", Run = "616163682", Jump = "616161997", Fall = "616157476", Climb = "616156119", Swim = "616165109", SwimIdle = "616166655" },
    Astronaut = { Idle1 = "891621366", Idle2 = "891633237", Walk = "891667138", Run = "891636393", Jump = "891627522", Fall = "891617961", Climb = "891609353", Swim = "891663592", SwimIdle = "891662115" },
    Ninja = { Idle1 = "656117400", Idle2 = "656118341", Walk = "656121766", Run = "656118852", Jump = "656117878", Fall = "656115606", Climb = "656114359", Swim = "656119721", SwimIdle = "656121397" },
    Mage = { Idle1 = "707742142", Idle2 = "707855907", Walk = "707897309", Run = "707861613", Jump = "707853694", Fall = "707829716", Climb = "707826056", Swim = "707876443", SwimIdle = "707894699" },
    Elder = { Idle1 = "845397899", Idle2 = "845400520", Walk = "845403856", Run = "845386501", Jump = "845398858", Fall = "845396048", Climb = "845392038", Swim = "845401742", SwimIdle = "845403127" },
    Toy = { Idle1 = "782841498", Idle2 = "782845736", Walk = "782843345", Run = "782842708", Jump = "782847020", Fall = "782846423", Climb = "782843869", Swim = "782844582", SwimIdle = "782845186" },
    Knight = { Idle1 = "657595757", Idle2 = "657568135", Walk = "657552124", Run = "657564596", Jump = "658409194", Fall = "657600338", Climb = "658360781", Swim = "657560551", SwimIdle = "657557095" },
    Cartoony = { Idle1 = "742637544", Idle2 = "742638445", Walk = "742640026", Run = "742638842", Jump = "742637942", Fall = "742637151", Climb = "742636889", Swim = "742639220", SwimIdle = "742639812" },
    SuperHero = { Idle1 = "616111295", Idle2 = "616113536", Walk = "616122287", Run = "616117076", Jump = "616115533", Fall = "616114533", Climb = "616104706", Swim = "616119360", SwimIdle = "616120861" },
    Bubbly = { Idle1 = "910004836", Idle2 = "910009958", Walk = "910034870", Run = "910025107", Jump = "910016857", Fall = "910001910", Climb = "909998997", Swim = "910028158", SwimIdle = "910030921" },
    Robot = { Idle1 = "616088211", Idle2 = "616089559", Walk = "616095330", Run = "616091570", Jump = "616090535", Fall = "616087089", Climb = "616086039", Swim = "616092998", SwimIdle = "616094091" },
    Stylish = { Idle1 = "616136790", Idle2 = "616138447", Walk = "616146177", Run = "616140816", Jump = "616139451", Fall = "616134815", Climb = "616133594", Swim = "616143378", SwimIdle = "616144772" },
    Confident = { Idle1 = "1069977950", Idle2 = "1069987858", Walk = "1070017263", Run = "1070001516", Jump = "1069984524", Fall = "1069973677", Climb = "1069946257", Swim = "1070009914", SwimIdle = "1070012133" },
    Popstar = { Idle1 = "1212900985", Idle2 = "1212900995", Walk = "1212980348", Run = "1212980338", Jump = "1212954642", Fall = "1212900995", Climb = "1213044953", Swim = "1213028260", SwimIdle = "1213028407" },
    Princess = { Idle1 = "941003647", Idle2 = "941013098", Walk = "941028902", Run = "941015281", Jump = "941008832", Fall = "941000007", Climb = "940996062", Swim = "941018893", SwimIdle = "941025398" },
	Fashionable = { Idle1 = "1243872664", Idle2 = "1243874093", Walk = "1243902226", Run = "1243892970", Jump = "1243880869", Fall = "1243876816", Climb = "1243878951", Swim = "1243893288", SwimIdle = "1243893659" },
	Cutesy = { Idle1 = "10921117521", Idle2 = "10921119966", Walk = "10921122459", Run = "10921121197", Jump = "10921122135", Fall = "10921121519", Climb = "10921123521", Swim = "10921124822", SwimIdle = "10921126038" },
	Bold = { Idle1 = "16738333868", Idle2 = "16738334710", Walk = "16738335850", Run = "16738336818", Jump = "16738337942", Fall = "16738339008", Climb = "16738340216", Swim = "16738341510", SwimIdle = "16738342679" },
	Glitch = { Idle1 = "16738344524", Idle2 = "16738345344", Walk = "16738346225", Run = "16738347097", Jump = "16738347980", Fall = "16738348824", Climb = "16738349687", Swim = "16738350561", SwimIdle = "16738351452" }
}

local famousEmotes = {
    {name = "Gangnam Style", id = "1843685019"},
    {name = "Thriller", id = "1843685126"},
    {name = "Macarena", id = "1843685280"},
    {name = "Running Man", id = "1843685432"},
    {name = "The Sprinkler", id = "1843685588"},
    {name = "Carlton Dance", id = "1843685741"},
    {name = "Moonwalk", id = "1843685894"},
    {name = "TikTok Dance", id = "5228907236"},
    {name = "Savage Love", id = "5230038335"},
    {name = "Say So Dance", id = "5230038432"},
    {name = "Renegade", id = "5230038520"},
    {name = "Marshmello Dance", id = "5230038611"},
    {name = "Dab", id = "3309406161"},
    {name = "Whip", id = "3309406274"},
    {name = "Nae Nae", id = "3309406377"},
    {name = "Hit The Quan", id = "3309406480"},
    {name = "Shoot Dance", id = "3309406589"},
    {name = "Passinho do Jamal", id = "1843685887"},
    {name = "Passinho do Faraó", id = "1843686025"},
    {name = "Passinho do Romano", id = "1843686173"},
    {name = "Passinho do Voltei", id = "1843686316"}
}

local currentEmoteTrack = nil
local emoteSpeed = 1

local function aplicarPack(pack)
    if not pack then return end
    
    local char = player.Character or player.CharacterAdded:Wait()
    local animate = char:WaitForChild("Animate")
    local humanoid = char:WaitForChild("Humanoid")
    local animator = humanoid:FindFirstChild("Animator")
    
    if animator then
        for _, track in pairs(animator:GetPlayingAnimationTracks()) do
            pcall(function() track:Stop() end)
        end
        
        local removed = 0
        for _, child in pairs(animator:GetChildren()) do
            if child:IsA("AnimationTrack") then
                pcall(function() child:Destroy() end)
                removed = removed + 1
            end
        end
        
        if removed > 0 then
            print(" Limpas " .. removed .. " tracks antigas")
        end
    end
    
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
    
    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    task.wait(0.1)
    humanoid:ChangeState(Enum.HumanoidStateType.Running)
end

local function playEmote(id, speed)
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    if currentEmoteTrack then
        currentEmoteTrack:Stop()
        currentEmoteTrack = nil
    end
    
    local animator = humanoid:FindFirstChild("Animator")
    if animator then
        for _, track in pairs(animator:GetPlayingAnimationTracks()) do
            pcall(function() track:Stop() end)
        end
        
        local tracksToRemove = {}
        for _, child in pairs(animator:GetChildren()) do
            if child:IsA("AnimationTrack") then
                table.insert(tracksToRemove, child)
            end
        end
        
        if #tracksToRemove > 50 then
            for i = 1, #tracksToRemove - 40 do
                pcall(function() tracksToRemove[i]:Destroy() end)
            end
            print(" Limpeza automática de tracks")
        end
    end
    
    local success = pcall(function()
        humanoid:PlayEmote(tostring(id))
    end)
    
    if success then
        print(" Emote oficial: " .. id)
        return
    end
    
    if not animator then
        animator = Instance.new("Animator")
        animator.Parent = humanoid
    end
    
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://" .. tostring(id)
    
    local track = animator:LoadAnimation(anim)
    if track then
        track.Priority = Enum.AnimationPriority.Action4
        track:Play()
        
        if speed and speed ~= 1 then
            track:AdjustSpeed(speed)
        end
        
        currentEmoteTrack = track
        print(" Animation: " .. id)
    else
        warn(" Falhou: " .. id)
        Notify("Emote não encontrado!", "error")
    end
end

local function stopEmote()
    if currentEmoteTrack then
        currentEmoteTrack:Stop()
        currentEmoteTrack = nil
    end
end

-- ================================ ESP HIGHLIGHT ================================

local ESPHighlights = {}
local lastESPUpdate = 0
local espRunning = false
local espLoopConn = nil

local function limparESPHighlight()
    for player, data in pairs(ESPHighlights) do
        pcall(function()
            if data.Highlight then data.Highlight:Destroy() end
            if data.Billboard then data.Billboard:Destroy() end
        end)
    end
    ESPHighlights = {}
end

local function atualizarESPHighlight()
    if not (ESPHighlightEnabled or ESPMM2HighlightEnabled) then
        if next(ESPHighlights) then limparESPHighlight() end
        return
    end
    
    if tick() - lastESPUpdate < 0.2 then return end
    lastESPUpdate = tick()
    
    local myChar = GetChar()
    local myHRP = myChar and GetHRP(myChar)
    if not myHRP then return end
    
    local myPos = myHRP.Position
    
    for _, plr in pairs(Players:GetPlayers()) do
        if plr == player then continue end
        
        local char = plr.Character
        local humanoid = char and GetHumanoid(char)
        
        if not char or not humanoid or humanoid.Health <= 0 then
            if ESPHighlights[plr] then
                pcall(function()
                    if ESPHighlights[plr].Highlight then ESPHighlights[plr].Highlight:Destroy() end
                    if ESPHighlights[plr].Billboard then ESPHighlights[plr].Billboard:Destroy() end
                end)
                ESPHighlights[plr] = nil
            end
            continue
        end
        
        local root = GetHRP(char)
        if not root then continue end
        
        local distancia = (root.Position - myPos).Magnitude
        
        if distancia > 300 then
            if ESPHighlights[plr] then
                pcall(function()
                    if ESPHighlights[plr].Highlight then ESPHighlights[plr].Highlight:Destroy() end
                    if ESPHighlights[plr].Billboard then ESPHighlights[plr].Billboard:Destroy() end
                end)
                ESPHighlights[plr] = nil
            end
            continue
        end
        
        local cor = Color3.fromRGB(255, 255, 0)
        
        if ESPMM2HighlightEnabled then
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
            local highlight = Instance.new("Highlight")
            highlight.Parent = char
            highlight.FillTransparency = 0.7
            highlight.OutlineTransparency = 0.3
            highlight.FillColor = cor
            highlight.OutlineColor = cor
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "ESPBillboard"
            billboard.Parent = char:FindFirstChild("Head") or char
            billboard.Size = UDim2.new(0, 100, 0, 30)
            billboard.StudsOffset = Vector3.new(0, 2.5, 0)
            billboard.AlwaysOnTop = true
            
            local textLabel = Instance.new("TextLabel")
            textLabel.Parent = billboard
            textLabel.Size = UDim2.new(1, 0, 1, 0)
            textLabel.BackgroundTransparency = 1
            textLabel.TextColor3 = cor
            textLabel.TextStrokeTransparency = 0.5
            textLabel.Font = Enum.Font.GothamBold
            textLabel.TextSize = 10
            textLabel.Text = plr.Name .. " [" .. math.floor(distancia) .. "m]"
            textLabel.TextScaled = true
            
            ESPHighlights[plr] = {Highlight = highlight, Billboard = billboard, TextLabel = textLabel}
        else
            local data = ESPHighlights[plr]
            if data.Highlight then
                data.Highlight.FillColor = cor
                data.Highlight.OutlineColor = cor
            end
            if data.TextLabel then
                data.TextLabel.Text = plr.Name .. " [" .. math.floor(distancia) .. "m]"
                data.TextLabel.TextColor3 = cor
            end
        end
    end
end

local function iniciarESPHighlight()
    if espRunning then return end
    espRunning = true
    
    if espLoopConn then espLoopConn:Disconnect() end
    espLoopConn = RunService.Heartbeat:Connect(function()
        atualizarESPHighlight()
    end)
end

local function pararESPHighlight()
    espRunning = false
    if espLoopConn then
        espLoopConn:Disconnect()
        espLoopConn = nil
    end
    limparESPHighlight()
end

-- ================================ ESP ARMA (OTIMIZADO) ================================

local espArmaActive = false

local function criarESPArmas()
    local myChar = GetChar()
    local myHRP = myChar and GetHRP(myChar)
    if not myHRP then return end
    
    local myPos = myHRP.Position
    
    for _, obj in pairs(workspace:GetDescendants()) do
        local isGun = false
        local nome = obj.Name:lower()
        local objPos = nil
        
        if obj:IsA("BasePart") then
            objPos = obj.Position
        elseif obj:FindFirstChild("Handle") and obj.Handle:IsA("BasePart") then
            objPos = obj.Handle.Position
        elseif obj:IsA("Model") and obj.PrimaryPart then
            objPos = obj.PrimaryPart.Position
        end
        
        if not objPos then continue end
        
        local distancia = (objPos - myPos).Magnitude
        if distancia > 300 then continue end  
        
        if obj:IsA("Tool") then
            if nome:find("gun") or nome:find("pistol") or nome:find("revolver") then
                isGun = true
            end
        end
        
        if not isGun and (obj:IsA("Model") or obj:IsA("MeshPart") or obj:IsA("Part")) then
            local hasGreenEffect = false
            for _, child in pairs(obj:GetChildren()) do
                if child:IsA("ParticleEmitter") and child.Enabled then
                    hasGreenEffect = true
                end
                if child:IsA("Highlight") and child.FillColor == Color3.fromRGB(0, 255, 0) then
                    hasGreenEffect = true
                end
            end
            
            if nome:find("gun") or nome:find("pistol") or nome:find("revolver") or nome:find("weapon") then
                isGun = true
            end
            
            if hasGreenEffect then
                isGun = true
            end
        end
        
        if not isGun and obj:FindFirstChild("Handle") then
            local handleName = obj.Handle.Name:lower()
            if handleName:find("gun") or handleName:find("pistol") or handleName:find("revolver") then
                isGun = true
            end
        end
        
        if isGun then
            local parent = obj.Parent
            local isHeld = false
            
            if parent and parent:IsA("Model") and parent:FindFirstChild("Humanoid") then
                local humanoid = parent:FindFirstChild("Humanoid")
                if humanoid and humanoid.Health > 0 then
                    isHeld = true
                end
            end
            
            if not isHeld and parent ~= player.Character then
                if not obj:FindFirstChild("ESPArmaHighlight") then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "ESPArmaHighlight"
                    highlight.Parent = obj
                    highlight.FillTransparency = 0.3
                    highlight.OutlineTransparency = 0.2
                    highlight.FillColor = Color3.fromRGB(255, 200, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 100, 0)
                    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                end
            end
        end
    end
end

local function removerESPArmas()
    for _, obj in pairs(workspace:GetDescendants()) do
        local highlight = obj:FindFirstChild("ESPArmaHighlight")
        if highlight then
            highlight:Destroy()
        end
    end
end

-- ================================ OCULTAR NOME ================================

local HideNameEnabled = false

local function aplicarOcultarNome(char)
    if not char then return end
    
    local humanoid = GetHumanoid(char)
    if humanoid then
        if HideNameEnabled then
            humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
        else
            humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.Viewer
        end
    end
end

local function aplicarEmTodos()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            aplicarOcultarNome(plr.Character)
        end
    end
end

EventManager:Add("HideNamePlayerAdded", Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        task.wait(0.5)
        aplicarOcultarNome(char)
    end)
end))

for _, plr in pairs(Players:GetPlayers()) do
    if plr ~= player then
        plr.CharacterAdded:Connect(function(char)
            task.wait(0.5)
            aplicarOcultarNome(char)
        end)
        aplicarOcultarNome(plr.Character)
    end
end

-- ================================ NOCLIP ================================

local NoClipEnabled = false

local function NoClipTask()
    if not NoClipEnabled then return end
    
    local char = GetChar()
    if char then
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end

-- ================================ AUTOFARM ================================

local AutoFarmEnabled = false
local AutoFarmNoClipActive = false
local AutoFarmMoving = false
local AutoFarmTween = nil

local function pararAutoFarmMovimento()
    if AutoFarmTween and AutoFarmTween.PlaybackState == Enum.PlaybackState.Playing then
        AutoFarmTween:Cancel()
    end
    AutoFarmMoving = false
    AutoFarmTween = nil
    
    if AutoFarmNoClipActive then
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
        AutoFarmNoClipActive = false
    end
end

local function moverParaMoeda(moeda)
    local char = GetChar()
    if not char then return false end
    
    local hrp = GetHRP(char)
    if not hrp then return false end
    
    if not AutoFarmNoClipActive then
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        AutoFarmNoClipActive = true
    end
    
    local targetPos = moeda.Position + Vector3.new(0, 2, 0)
    local distancia = (targetPos - hrp.Position).Magnitude
    local velocidade = 45
    local duracao = distancia / velocidade
    
    if duracao < 0.5 then duracao = 0.5 end
    
    local tweenInfo = TweenInfo.new(duracao, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
    local tween = TweenService:Create(hrp, tweenInfo, {CFrame = CFrame.new(targetPos)})
    
    AutoFarmTween = tween
    AutoFarmMoving = true
    tween:Play()
    tween.Completed:Wait()
    task.wait(0.2)
    AutoFarmMoving = false
    
    return true
end

local function getNearestCoinNatural()
    local myHRP = GetHRP()
    if not myHRP then return nil end
    
    atualizarCacheMoedas()
    
    local nearestCoin = nil
    local nearestDist = 150 -- REDUZIDO para mobile
    
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
    if not AutoFarmEnabled then return end
    if AutoFarmMoving then return end
    
    local moeda = getNearestCoinNatural()
    if moeda then
        moverParaMoeda(moeda)
    end
end

local function AutoFarmToggle(Value)
    AutoFarmEnabled = Value
    
    if Value then
        LoopManager:AddThrottled("AutoFarm", AutoFarmTaskNatural, 0.2)
    else
        LoopManager:RemoveThrottled("AutoFarm")
        pararAutoFarmMovimento()
    end
end

-- ================================ AIMBOT ================================

local AimbotEnabled = false
local SelectedAimPart = "Head"
local AimbotFOV = 90
local AimbotSmoothness = 5
local IgnoreWalls = false
local lastPositions = {}
local lastPositionsCount = 0
local FOVCircle = nil
local FOVCircleFrame = nil

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
    if result then
        return result.Instance:IsDescendantOf(target.Character)
    end
    
    return true
end

local function UpdateFOVCircle()
    if not AimbotEnabled then
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
        stroke.Color = Color3.fromRGB(220, 50, 50)
        stroke.Thickness = 0.3
        stroke.Transparency = 0
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(1, 0)
        corner.Parent = circle
        
        FOVCircle = gui
        FOVCircleFrame = circle
    end
    
    if FOVCircleFrame then
        FOVCircleFrame.Size = UDim2.new(0, AimbotFOV * 2, 0, AimbotFOV * 2)
        FOVCircleFrame.Position = UDim2.new(0.5, -AimbotFOV, 0.5, -AimbotFOV)
    end
end

local function GetClosestPlayerWithFOV(ignoreWalls)
    local closestPlayer = nil
    local bestScore = math.huge
    
    local myPos = GetChar() and GetChar():FindFirstChild("Head") and GetChar().Head.Position
    if not myPos then return nil end
    
    atualizarPlayerCache()
    
    for _, otherPlayer in pairs(playerCache) do
        local char = otherPlayer.Character
        if not char then continue end
        
        local aimPart
        if SelectedAimPart == "Head" then
            aimPart = char:FindFirstChild("Head")
        else
            aimPart = GetHRP(char)
        end
        if not aimPart then continue end
        
        local humanoid = GetHumanoid(char)
        if not humanoid or humanoid.Health <= 0 then continue end
        
        if ignoreWalls and not CanSeePlayer(otherPlayer) then continue end
        
        local targetPos = aimPart.Position
        
        if lastPositionsCount > 100 then
            local oldest = nil
            local oldestTime = math.huge
            for k, v in pairs(lastPositions) do
                if v.time < oldestTime then
                    oldestTime = v.time
                    oldest = k
                end
            end
            if oldest then
                lastPositions[oldest] = nil
                lastPositionsCount = lastPositionsCount - 1
            end
        end
        
        local velocity = Vector3.new(0, 0, 0)
        if lastPositions[otherPlayer] then
            local lastPos, lastTime = lastPositions[otherPlayer].pos, lastPositions[otherPlayer].time
            local dt = tick() - lastTime
            if dt > 0 then
                velocity = (targetPos - lastPos) / dt
            end
        end
        
        lastPositions[otherPlayer] = {pos = targetPos, time = tick()}
        lastPositionsCount = lastPositionsCount + 1
        
        local predictedPos = targetPos + (velocity * 0.2)
        local screenPoint = camera:WorldToViewportPoint(predictedPos)
        if screenPoint.Z <= 0 then continue end
        
        local centerDist = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)).Magnitude
        if centerDist > AimbotFOV then continue end
        
        local realDist = (targetPos - myPos).Magnitude
        local score = (centerDist * 0.7) + (realDist * 0.3)
        
        if score < bestScore then
            bestScore = score
            closestPlayer = otherPlayer
        end
    end
    
    return closestPlayer
end

local function AimbotTask()
    if not AimbotEnabled then return end
    if GetChar() and GetHRP() then
        local target = GetClosestPlayerWithFOV(IgnoreWalls)
        if target and target.Character then
            local aimPart
            if SelectedAimPart == "Head" then
                aimPart = target.Character:FindFirstChild("Head")
            else
                aimPart = GetHRP(target.Character)
            end
            
            if aimPart then
                local targetPos = aimPart.Position
                if lastPositions[target] then
                    local lastPos, lastTime = lastPositions[target].pos, lastPositions[target].time
                    local dt = tick() - lastTime
                    if dt > 0 then
                        local velocity = (targetPos - lastPos) / dt
                        targetPos = targetPos + (velocity * 0.15)
                    end
                end
                
                local currentLook = camera.CFrame.LookVector
                local targetLook = (targetPos - camera.CFrame.Position).Unit
                local distancia = (targetPos - camera.CFrame.Position).Magnitude
                local smoothFactor = 1 / (AimbotSmoothness * (1 + distancia / 100))
                local smoothLook = currentLook:Lerp(targetLook, smoothFactor)
                
                camera.CFrame = CFrame.lookAt(camera.CFrame.Position, camera.CFrame.Position + smoothLook)
            end
        end
    end
end

-- ================================ VARIÁVEIS ================================

local LockCamEnabled = false
local ShowFPSEnabled = false
local FPSUnlocked = false
local backpackActive = false
local headsitActive = false
local spectateActive = false
local infJumpEnabled = false
local flingOPActive = false
local LockUIActive = false
local SpeedToggleEnabled = false
local FOVCamToggleEnabled = false
local SpinbotEnabled = false
local headlessActive = false
local korbloxActive = false
local flying = false
local ESPHighlightEnabled = false
local ESPMM2HighlightEnabled = false

local SpeedValue = 16
local FovCamValue = 70
local FlySpeed = 50

local selectedPlayer = nil
local jumpConnection = nil
local posicaoOriginal = nil
local savedPosition = nil
local lastPositionTP = nil
local originalHead = nil
local originalLeftLeg = nil
local bodyVelocity = nil
local bodyGyro = nil
local playerDropdown = nil
local selectedAnimationPack = nil

local LockUI
local FPSUI
local MiraUI

local killAssassinActive = false
local killSheriffActive = false
local killAssassinConnection = nil
local killSheriffConnection = nil
local killAssassinPos = nil
local killSheriffPos = nil

-- ================================ FUNÇÕES DE VELOCIDADE E CAM ================================

local function SpeedTask()
    local char = GetChar()
    if char then
        local humanoid = GetHumanoid(char)
        if humanoid then
            humanoid.WalkSpeed = SpeedValue
        end
    end
end

local function FovCamTask()
    if camera then
        camera.FieldOfView = FovCamValue
    end
end

local function LockCamTask()
    if LockCamEnabled and LockUIActive then
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

-- ================================ UTILIDADES ================================

local function CriarFPSDisplay()
    local gui = Instance.new("ScreenGui")
    gui.Name = "FPSDisplay"
    gui.Parent = player:FindFirstChild("PlayerGui") or player:WaitForChild("PlayerGui")
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 999
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Enabled = false
    
    local frame = Instance.new("Frame")
    frame.Parent = gui
    frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    frame.BackgroundTransparency = 0.5
    frame.Size = UDim2.new(0, 70, 0, 25)
    frame.Position = UDim2.new(1, -80, 0, 8)
    frame.Active = false
    frame.Draggable = false
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = frame
    
    local fpsLabel = Instance.new("TextLabel")
    fpsLabel.Parent = frame
    fpsLabel.BackgroundTransparency = 1
    fpsLabel.Size = UDim2.new(1, 0, 1, 0)
    fpsLabel.Text = "FPS: 60"
    fpsLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
    fpsLabel.TextScaled = true
    fpsLabel.Font = Enum.Font.GothamBold
    fpsLabel.TextStrokeTransparency = 0.5
    
    local frameCount = 0
    local lastTime = tick()
    local fps = 60
    
    local renderConn = RunService.RenderStepped:Connect(function()
        if not gui.Enabled then return end
        
        frameCount = frameCount + 1
        local currentTime = tick()
        
        if currentTime - lastTime >= 1 then
            fps = frameCount
            frameCount = 0
            lastTime = currentTime
            
            if fps >= 50 then
                fpsLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
            elseif fps >= 30 then
                fpsLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
            else
                fpsLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
            end
            
            fpsLabel.Text = "FPS: " .. fps
        end
    end)
    
    gui.AncestryChanged:Connect(function()
        if not gui.Parent then
            renderConn:Disconnect()
        end
    end)
    
    return gui
end

FPSUI = CriarFPSDisplay()

-- ================================ PULO INFINITO ================================

jumpConnection = UserInputService.JumpRequest:Connect(function()
    if infJumpEnabled and not UserInputService:GetFocusedTextBox() then
        local humanoid = GetChar() and GetChar():FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:ChangeState("Jumping")
        end
    end
end)

-- ================================ HUD ================================

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
        LockUIActive = active
    end
    
    button.MouseButton1Click:Connect(function()
        local novoEstado = not LockUIActive
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

-- ================================ FLY ================================

local function startFly()
    local char = GetChar()
    if not char then
        Notify("Personagem não encontrado", "error")
        return
    end
    
    local humanoid = GetHumanoid(char)
    local hrp = GetHRP(char)
    
    if not humanoid or not hrp then
        Notify("Humanoid ou HRP não encontrado", "error")
        return
    end
    
    if bodyVelocity then pcall(function() bodyVelocity:Destroy() end) end
    if bodyGyro then pcall(function() bodyGyro:Destroy() end) end
    
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Name = "FlyVelocity"
    bodyVelocity.MaxForce = Vector3.new(999999, 999999, 999999)
    bodyVelocity.Parent = hrp
    
    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.Name = "FlyGyro"
    bodyGyro.MaxTorque = Vector3.new(999999, 999999, 999999)
    bodyGyro.Parent = hrp
    
    humanoid.PlatformStand = true
    
    LoopManager:AddFrame("FlyUpdate", function()
        if not flying then return end
        
        local char = GetChar()
        if not char then
            if flying then
                flying = false
                StopFly()
            end
            return
        end
        
        local humanoid = GetHumanoid(char)
        local hrp = GetHRP(char)
        
        if not humanoid or not hrp then
            return
        end
        
        local camera = workspace.CurrentCamera
        local moveDir = humanoid.MoveDirection
        
        local velocity = moveDir * FlySpeed
        
        if moveDir.Magnitude > 0 then
            local lookY = camera.CFrame.LookVector.Y
            local forwardDot = moveDir:Dot(camera.CFrame.LookVector)
            local directionFix = (forwardDot >= 0) and 1 or -1
            velocity = velocity + Vector3.new(0, lookY * FlySpeed * directionFix, 0)
        end
        
        if bodyVelocity then
            bodyVelocity.Velocity = velocity
        end
        
        if bodyGyro then
            bodyGyro.CFrame = camera.CFrame
        end
    end)
end

local function StopFly()
    flying = false
    
    local char = GetChar()
    if char then
        local humanoid = GetHumanoid(char)
        if humanoid then
            humanoid.PlatformStand = false
        end
    end
    
    if bodyVelocity then
        pcall(function() bodyVelocity:Destroy() end)
        bodyVelocity = nil
    end
    
    if bodyGyro then
        pcall(function() bodyGyro:Destroy() end)
        bodyGyro = nil
    end
    
    LoopManager:RemoveFrame("FlyUpdate")
end

local function toggleFly(state)
    flying = state
    if state then
        startFly()
    else
        StopFly()
    end
end

-- ================================ SPINBOT ================================

local function toggleSpinbot(state)
    SpinbotEnabled = state
    
    if state then
        LoopManager:AddFrame("Spinbot", function()
            if not SpinbotEnabled then return end
            local char = GetChar()
            if char then
                local hrp = GetHRP(char)
                if hrp then
                    hrp.RotVelocity = Vector3.new(0, 200, 0)
                end
            end
        end)
    else
        LoopManager:RemoveFrame("Spinbot")
        local char = GetChar()
        if char then
            local hrp = GetHRP(char)
            if hrp then
                hrp.RotVelocity = Vector3.new(0, 0, 0)
            end
        end
    end
end

-- ================================ FUNÇÕES DE LISTA ================================

local function getPlayerList()
    local list = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player then
            table.insert(list, p.Name)
        end
    end
    return list
end

local function stopAll()
    if backpackActive then
        backpackActive = false
        LoopManager:RemoveFrame("Backpack")
    end
    
    if headsitActive then
        headsitActive = false
        LoopManager:RemoveFrame("Headsit")
    end
    
    if spectateActive then
        spectateActive = false
        local myChar = GetChar()
        if myChar and myChar:FindFirstChild("Humanoid") then
            camera.CameraSubject = myChar.Humanoid
            camera.CameraType = Enum.CameraType.Custom
        end
    end
    
    if flingOPActive then
        flingOPActive = false
        LoopManager:RemoveFrame("FlingOP")
        
        local meuHRP = GetHRP()
        if meuHRP then
            meuHRP.RotVelocity = Vector3.new(0, 0, 0)
            meuHRP.Velocity = Vector3.new(0, 0, 0)
            meuHRP.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            meuHRP.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            
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

-- ================================ FUNÇÕES DE HEAD/KORBLOX ================================

local function toggleHeadless(state)
    local char = GetChar()
    if not char then return end

    local head = char:FindFirstChild("Head")
    if not head then return end

    local hairKeywords = {"hair", "cabelo", "wig", "cabel", "hairstyle", "bangs", "ponytail", "braid", "haircut"}
    
    local function isHair(accessoryName)
        local lowerName = accessoryName:lower()
        for _, keyword in pairs(hairKeywords) do
            if lowerName:find(keyword) then
                return true
            end
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
        if face then
            face.Transparency = 1
        end

        for _, accessory in pairs(char:GetChildren()) do
            if accessory:IsA("Accessory") then
                local handle = accessory:FindFirstChild("Handle")
                local weld = handle and handle:FindFirstChildOfClass("Weld")
                
                if isHair(accessory.Name) then
                    if handle then
                        handle.Transparency = 0
                    end
                else
                    if weld and weld.Part1 == head then
                        if handle then
                            handle.Transparency = 1
                        end
                    end
                end
            end
        end
    else
        head.Transparency = 0
        head.CanCollide = true
        head.Massless = false

        local face = head:FindFirstChildOfClass("Decal")
        if face then
            face.Transparency = 0
        end

        for _, accessory in pairs(char:GetChildren()) do
            if accessory:IsA("Accessory") then
                local handle = accessory:FindFirstChild("Handle")
                if handle then
                    handle.Transparency = 0
                end
            end
        end

        if originalHead then
            originalHead:Destroy()
            originalHead = nil
        end
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

local function updateVisuals()
    toggleHeadless(headlessActive)
    toggleKorblox(korbloxActive)
end

-- ================================ EVENTOS CENTRALIZADOS ================================

EventManager:Add("CharacterAdded", player.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    
    local humanoid = GetHumanoid(char)
    if humanoid then
        local animator = humanoid:FindFirstChild("Animator")
        if animator then
            for _, track in pairs(animator:GetPlayingAnimationTracks()) do
                pcall(function() track:Stop() end)
            end
        end
    end
    
    if flying then
        startFly()
    end
    
    if headlessActive or korbloxActive then
        updateVisuals()
    end
    
    if selectedAnimationPack then
        aplicarPack(ANIMATION_PACKS[selectedAnimationPack])
    end
end))

EventManager:Add("CharacterRemoving", player.CharacterRemoving:Connect(function()
    if bodyVelocity then
        pcall(function() bodyVelocity:Destroy() end)
        bodyVelocity = nil
    end
    if bodyGyro then
        pcall(function() bodyGyro:Destroy() end)
        bodyGyro = nil
    end
end))

-- ================================ CRIAÇÃO DAS TABS ================================

local AimbotTab = Window:Tab({Title = "Aimbot", Icon = "crosshair"})

AimbotTab:Toggle({
    Title = "Ativar Aimbot",
    Default = false,
    Callback = function(Value)
        AimbotEnabled = Value
        if AimbotEnabled then
            LoopManager:AddFrame("Aimbot", AimbotTask)
        else
            LoopManager:RemoveFrame("Aimbot")
        end
        UpdateFOVCircle()
    end
})

AimbotTab:Dropdown({
    Title = "Target",
    Values = {"Head", "Torso"},
    Default = "Head",
    Callback = function(Value)
        SelectedAimPart = Value
    end
})

AimbotTab:Slider({
    Title = "FOV",
    Step = 1,
    Value = {
        Min = 30,
        Max = 150,
        Default = 90,
    },
    Callback = function(Value)
        AimbotFOV = Value
        UpdateFOVCircle()
    end
})

AimbotTab:Slider({
    Title = "Smoothing",
    Step = 1,
    Value = {
        Min = 1,
        Max = 10,
        Default = 5,
    },
    Callback = function(Value)
        AimbotSmoothness = Value
    end
})

AimbotTab:Toggle({
    Title = "Ignore Walls",
    Default = false,
    Callback = function(Value)
        IgnoreWalls = Value
    end
})

-- ================================ ABA VISUAIS ================================

local VisualsTab = Window:Tab({Title = "Visuais", Icon = "eye"})

VisualsTab:Toggle({
    Title = "Enabled ESP",
    Default = false,
    Callback = function(Value)
        if Value then
            if ESPMM2HighlightEnabled then
                Notify("Desative o ESP MM2 primeiro", "error")
                return
            end
            ESPHighlightEnabled = true
            iniciarESPHighlight()  
        else
            ESPHighlightEnabled = false
            if not ESPMM2HighlightEnabled then
                pararESPHighlight()  
            end
        end
    end
})

VisualsTab:Toggle({
    Title = "Hide @User (Clean Names)",
    Default = false,
    Callback = function(Value)
        HideNameEnabled = Value
        aplicarEmTodos()
    end
})

VisualsTab:Toggle({
    Title = "Show FPS",
    Default = false,
    Callback = function(Value)
        ShowFPSEnabled = Value
        if FPSUI then
            FPSUI.Enabled = Value
        end
    end
})

VisualsTab:Section({Title = "Hub Cosmetics"})

VisualsTab:Toggle({
    Title = "Headless (Visual Only)",
    Default = false,
    Callback = function(Value)
        headlessActive = Value
        toggleHeadless(Value)
    end
})

VisualsTab:Toggle({
    Title = "Korblox (Visual Only)",
    Default = false,
    Callback = function(Value)
        korbloxActive = Value
        toggleKorblox(Value)
    end
})

-- ================================ ABA JOGADOR ================================

local JogadorTab = Window:Tab({Title = "Exploit", Icon = "box"})

JogadorTab:Section({Title = "Movement & Tools"})

JogadorTab:Toggle({
    Title = "Fly",
    Default = false,
    Callback = function(Value)
        toggleFly(Value)
    end
})

JogadorTab:Slider({
    Title = "Fly Speed",
    Step = 1,
    Value = {
        Min = 10,
        Max = 300,
        Default = 50,
    },
    Callback = function(Value)
        FlySpeed = Value
    end
})

JogadorTab:Toggle({
    Title = "Speed",
    Default = false,
    Callback = function(Value)
        SpeedToggleEnabled = Value
        if Value then
            SpeedTask()
            LoopManager:AddFrame("Speed", SpeedTask)
        else
            local char = GetChar()
            if char then
                local humanoid = GetHumanoid(char)
                if humanoid then
                    humanoid.WalkSpeed = 16
                end
            end
            LoopManager:RemoveFrame("Speed")
        end
    end
})

JogadorTab:Slider({
    Title = "Speed Walk",
    Step = 1,
    Value = {
        Min = 16,
        Max = 150,
        Default = 16,
    },
    Callback = function(Value)
        SpeedValue = Value
        if SpeedToggleEnabled then
            SpeedTask()
        end
    end
})

JogadorTab:Toggle({
    Title = "FOV Cam",
    Default = false,
    Callback = function(Value)
        FOVCamToggleEnabled = Value
        if Value then
            FovCamTask()
            LoopManager:AddFrame("FovCam", FovCamTask)
        else
            if camera then
                camera.FieldOfView = 70
            end
            LoopManager:RemoveFrame("FovCam")
        end
    end
})

JogadorTab:Slider({
    Title = "FOV Cam (Zoom)",
    Step = 1,
    Value = {
        Min = 1,
        Max = 120,
        Default = 70,
    },
    Callback = function(Value)
        FovCamValue = Value
        if FOVCamToggleEnabled then
            FovCamTask()
        end
    end
})

JogadorTab:Toggle({
    Title = "Infinite Jump",
    Default = false,
    Callback = function(Value)
        infJumpEnabled = Value
    end
})

JogadorTab:Toggle({
    Title = "NoClip",
    Default = false,
    Callback = function(Value)
        NoClipEnabled = Value
        if Value then
            LoopManager:AddFrame("NoClip", NoClipTask)
        else
            LoopManager:RemoveFrame("NoClip")
            local char = GetChar()
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = true
                    end
                end
            end
        end
    end
})

JogadorTab:Section({Title = "Tools & Utilities"})

JogadorTab:Toggle({
    Title = "Spinbot",
    Default = false,
    Callback = function(Value)
        toggleSpinbot(Value)
    end
})

JogadorTab:Toggle({
    Title = "Lock Cam",
    Default = false,
    Callback = function(Value)
        LockCamEnabled = Value
        if LockCamEnabled then
            CreateShiftLockButton()
            CriarMira()
            LoopManager:AddFrame("LockCam", LockCamTask)
        else
            if LockUI then
                LockUI:Destroy()
                LockUI = nil
            end
            if MiraUI then
                MiraUI:Destroy()
                MiraUI = nil
            end
            LoopManager:RemoveFrame("LockCam")
            local char = GetChar()
            if char then
                local humanoid = GetHumanoid(char)
                if humanoid then
                    humanoid.CameraOffset = Vector3.new(0, 0, 0)
                end
            end
        end
    end
})

JogadorTab:Button({
    Title = "Click TP",
    Callback = function()
        local success, err = pcall(function()
            local plr = Players.LocalPlayer
            if not plr.Character then return end
            
            local tool = Instance.new("Tool")
            tool.Name = "TP Tool"
            tool.RequiresHandle = false
            tool.CanBeDropped = false
            tool.TextureId = "rbxassetid://13060727541"
            
            tool.Activated:Connect(function()
                local mouse = plr:GetMouse()
                local target = mouse.Target
                local hit = mouse.Hit
                
                if target and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local rayOrigin = hit.Position + Vector3.new(0, 5, 0)
                    local rayDirection = Vector3.new(0, -10, 0)
                    local raycastParams = RaycastParams.new()
                    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                    raycastParams.FilterDescendantsInstances = {plr.Character}
                    
                    local rayResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
                    
                    if rayResult and rayResult.Instance and rayResult.Instance.CanCollide then
                        plr.Character.HumanoidRootPart.CFrame = CFrame.new(rayResult.Position + Vector3.new(0, 3, 0))
                    elseif hit.Position.Y > 3 and hit.Position.Y < 1000 then
                        plr.Character.HumanoidRootPart.CFrame = CFrame.new(hit.Position + Vector3.new(0, 3, 0))
                    end
                end
            end)
            
            tool.Parent = plr.Backpack
        end)
        
        if not success then
            Notify("Erro ao criar TP Tool", "error")
        end
    end
})

JogadorTab:Button({
    Title = "Reiniciar",
    Callback = function()
        local char = GetChar()
        if not char then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local hrp = GetHRP(char)
        if not hrp then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        local posicaoAtual = hrp.CFrame
        local flyEstavaAtivo = flying
        
        if flyEstavaAtivo then
            toggleFly(false)
        end
        
        local humanoid = GetHumanoid(char)
        if humanoid then
            humanoid.Health = 0
        end
        
        local newChar = player.CharacterAdded:Wait()
        task.wait(0.1)
        
        local newHRP = GetHRP(newChar)
        if newHRP then
            newHRP.CFrame = posicaoAtual
        end
        
        if flyEstavaAtivo then
            task.wait(0.2)
            toggleFly(true)
        end
    end
})

-- ================================ ABA ONLINES ================================

local OnlinesTab = Window:Tab({Title = "Onlines", Icon = "users"})

OnlinesTab:Section({Title = "Jogadores"})

local lastUpdate = 0

local function updatePlayerDropdown()
    if not playerDropdown then return end
    if tick() - lastUpdate < 0.5 then return end
    lastUpdate = tick()
    
    local newList = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player then
            table.insert(newList, p.Name)
        end
    end
    playerDropdown:Refresh(newList)
end

playerDropdown = OnlinesTab:Dropdown({
    Title = "Selecionar Jogador",
    Values = getPlayerList(),
    Default = "Ninguem",
    Callback = function(name)
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Name == name and p ~= player then
                selectedPlayer = p
                break
            end
        end
    end
})

EventManager:Add("PlayerAdded", Players.PlayerAdded:Connect(function()
    updatePlayerDropdown()
end))

EventManager:Add("PlayerRemoving", Players.PlayerRemoving:Connect(function(p)
    if selectedPlayer == p then
        selectedPlayer = nil
        stopAll()
    end
    updatePlayerDropdown()
end))

OnlinesTab:Section({Title = "Ações"})

OnlinesTab:Button({
    Title = "Spectate",
    Callback = function()
        if not selectedPlayer then
            Notify("Selecione um jogador primeiro", "error")
            return
        end
        stopAll()
        
        local char = selectedPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            camera.CameraSubject = char.Humanoid
            camera.CameraType = Enum.CameraType.Custom
            spectateActive = true
        else
            Notify("Jogador sem personagem", "error")
        end
    end
})

OnlinesTab:Button({
    Title = "Teleport",
    Callback = function()
        if not selectedPlayer then
            Notify("Selecione um jogador primeiro", "error")
            return
        end
        stopAll()
        
        local targetChar = selectedPlayer.Character
        local myChar = GetChar()
        
        if targetChar and targetChar:FindFirstChild("HumanoidRootPart") and myChar and myChar:FindFirstChild("HumanoidRootPart") then
            myChar.HumanoidRootPart.CFrame = targetChar.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
        else
            Notify("Jogador sem personagem", "error")
        end
    end
})

OnlinesTab:Button({
    Title = "Mochila",
    Callback = function()
        if not selectedPlayer then
            Notify("Selecione um jogador primeiro", "error")
            return
        end
        
        if backpackActive then
            stopAll()
            Notify("Backpack desativado", "info")
            return
        end
        
        stopAll()
        backpackActive = true
        
        LoopManager:AddFrame("Backpack", function()
            if not backpackActive or not selectedPlayer or not selectedPlayer.Character then
                LoopManager:RemoveFrame("Backpack")
                return
            end
            
            local myChar = GetChar()
            if myChar and myChar:FindFirstChild("HumanoidRootPart") then
                local targetRoot = selectedPlayer.Character:FindFirstChild("HumanoidRootPart")
                if targetRoot then
                    myChar.HumanoidRootPart.CFrame = targetRoot.CFrame * CFrame.new(0, 0, 1.3) * CFrame.Angles(0, math.rad(180), 0)
                    if myChar.Humanoid:GetState() ~= Enum.HumanoidStateType.Seated then
                        myChar.Humanoid.Sit = true
                    end
                end
            end
        end)
    end
})

OnlinesTab:Button({
    Title = "Headsit",
    Callback = function()
        if not selectedPlayer then
            Notify("Selecione um jogador primeiro", "error")
            return
        end
        
        if headsitActive then
            stopAll()
            Notify("Headsit desativado", "info")
            return
        end
        
        stopAll()
        headsitActive = true
        
        LoopManager:AddFrame("Headsit", function()
            if not headsitActive or not selectedPlayer or not selectedPlayer.Character then
                LoopManager:RemoveFrame("Headsit")
                return
            end
            
            local myChar = GetChar()
            if myChar and myChar:FindFirstChild("HumanoidRootPart") then
                local targetHead = selectedPlayer.Character:FindFirstChild("Head")
                local targetRoot = selectedPlayer.Character:FindFirstChild("HumanoidRootPart")
                
                if targetHead and targetRoot then
                    local headPos = targetHead.Position + Vector3.new(0, 2.5, 0)
                    local lookDirection = targetRoot.CFrame.LookVector
                    lookDirection = Vector3.new(lookDirection.X, 0, lookDirection.Z).Unit
                    
                    myChar.HumanoidRootPart.CFrame = CFrame.lookAt(headPos, headPos + lookDirection)
                    
                    if myChar.Humanoid:GetState() ~= Enum.HumanoidStateType.Seated then
                        myChar.Humanoid.Sit = true
                    end
                end
            end
        end)
    end
})

OnlinesTab:Button({
    Title = "Fling OP",
    Callback = function()
        if not selectedPlayer then
            Notify("Selecione um jogador primeiro", "error")
            return
        end
        
        if flingOPActive then
            stopAll()
            Notify("Fling OP desativado", "info")
            return
        end
        
        stopAll()
        
        local targetChar = selectedPlayer.Character
        local myChar = GetChar()
        
        if not targetChar or not myChar then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local targetHRP = GetHRP(targetChar)
        local myHRP = GetHRP(myChar)
        
        if not targetHRP or not myHRP then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        flingOPActive = true
        posicaoOriginal = myHRP.CFrame
        
        LoopManager:AddFrame("FlingOP", function()
            if not flingOPActive then
                LoopManager:RemoveFrame("FlingOP")
                return
            end
            
            if not selectedPlayer or not selectedPlayer.Character then
                stopAll()
                return
            end
            
            local myHRP = GetHRP()
            local targetHRP = GetHRP(selectedPlayer.Character)
            
            if not myHRP or not targetHRP then
                return
            end
            
            myHRP.RotVelocity = Vector3.new(
                math.random(8000, 9000),
                math.random(8000, 9000),
                math.random(8000, 9000)
            )
            
            local randomAngle = CFrame.Angles(
                math.rad(math.random(-360, 360)),
                math.rad(math.random(-360, 360)),
                math.rad(math.random(-360, 360))
            )
            
            local yOffset = math.random(-6, 6)
            
            myHRP.CFrame = targetHRP.CFrame *
                CFrame.new(
                    math.random(-1, 1),
                    yOffset,
                    math.random(-1, 1)
                ) * randomAngle
            
            myHRP.Velocity = Vector3.new(
                math.random(-500, 500),
                math.random(400, 900),
                math.random(-500, 500)
            )
        end)
    end
})

OnlinesTab:Button({
    Title = "Stop",
    Callback = function()
        local temAcaoAtiva = backpackActive or headsitActive or spectateActive or flingOPActive        
        if not temAcaoAtiva then
            Notify("Nenhuma ação ativa na aba Onlines para parar!", "info")
            return
        end
        stopAll()
    end
})

-- ================================ ABA MURDER MYSTERY 2 ================================

local MM2Tab = Window:Tab({Title = "Murder Mystery 2", Icon = "axe"})

MM2Tab:Section({Title = "Visuais"})

MM2Tab:Toggle({
    Title = "ESP de Jogadores",
    Default = false,
    Callback = function(Value)
        if Value then
            if ESPHighlightEnabled then
                Notify("Desative o ESP Visual primeiro", "error")
                return
            end
            ESPMM2HighlightEnabled = true
            iniciarESPHighlight()
        else
            ESPMM2HighlightEnabled = false
            if not ESPHighlightEnabled then
                pararESPHighlight()
            end
        end
    end
})

MM2Tab:Toggle({
    Title = "ESP Arma",
    Default = false,
    Callback = function(Value)
        espArmaActive = Value
        if Value then
            criarESPArmas()
            LoopManager:AddThrottled("ESPArmas", function()
                if espArmaActive then
                    criarESPArmas()
                end
            end, 1)
        else
            LoopManager:RemoveThrottled("ESPArmas")
            removerESPArmas()
        end
    end
})

local function encontrarPorPapel(papelAlvo)
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            if getPapel(plr) == papelAlvo then
                return plr
            end
        end
    end
    return nil
end

MM2Tab:Section({Title = "Combat"})

MM2Tab:Button({
    Title = "Matar Xerife",
    Callback = function()
        local xerife = encontrarPorPapel("Xerife")
        if xerife and xerife.Character then
            local targetHRP = xerife.Character:FindFirstChild("HumanoidRootPart")
            local myHRP = GetHRP()
            if targetHRP and myHRP then
                targetHRP.CFrame = myHRP.CFrame
            end
        else
            Notify("Xerife nao encontrado", "error")
        end
    end
})

MM2Tab:Button({
    Title = "Matar Todos",
    Callback = function()
        local myHRP = GetHRP()
        if not myHRP then return end
        
        local myPos = myHRP.Position
        
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local targetHRP = plr.Character:FindFirstChild("HumanoidRootPart")
                if targetHRP then
                    local distancia = (targetHRP.Position - myPos).Magnitude
                    if distancia <= 300 then
                        targetHRP.CFrame = myHRP.CFrame
                        task.wait(0.1)
                    end
                end
            end
        end
    end
})

local function pararJogarAssassino()
    if killAssassinActive then
        killAssassinActive = false
        if killAssassinConnection then
            killAssassinConnection:Disconnect()
            killAssassinConnection = nil
        end
        LoopManager:RemoveFrame("KillAssassinFling")
        local meuHRP = GetHRP()
        if meuHRP then
            meuHRP.RotVelocity = Vector3.new(0, 0, 0)
            meuHRP.Velocity = Vector3.new(0, 0, 0)
            meuHRP.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            meuHRP.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            if killAssassinPos then
                meuHRP.CFrame = killAssassinPos
                killAssassinPos = nil
            end
        end
    end
end

local function pararJogarXerife()
    if killSheriffActive then
        killSheriffActive = false
        if killSheriffConnection then
            killSheriffConnection:Disconnect()
            killSheriffConnection = nil
        end
        LoopManager:RemoveFrame("KillSheriffFling")
        local meuHRP = GetHRP()
        if meuHRP then
            meuHRP.RotVelocity = Vector3.new(0, 0, 0)
            meuHRP.Velocity = Vector3.new(0, 0, 0)
            meuHRP.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            meuHRP.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            if killSheriffPos then
                meuHRP.CFrame = killSheriffPos
                killSheriffPos = nil
            end
        end
    end
end

MM2Tab:Button({
    Title = "Jogar Assassino Longe",
    Callback = function()
        if killAssassinActive then
            pararJogarAssassino()
            return
        end
        
        local assassino = encontrarPorPapel("Assassino")
        if not assassino then
            Notify("Nenhum assassino encontrado", "error")
            return
        end
        
        local targetChar = assassino.Character
        local myChar = GetChar()
        
        if not targetChar or not myChar then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local targetHRP = GetHRP(targetChar)
        local myHRP = GetHRP(myChar)
        
        if not targetHRP or not myHRP then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        killAssassinActive = true
        killAssassinPos = myHRP.CFrame
        
        LoopManager:AddFrame("KillAssassinFling", function()
            if not killAssassinActive then
                LoopManager:RemoveFrame("KillAssassinFling")
                return
            end
            
            local currentAssassino = encontrarPorPapel("Assassino")
            if not currentAssassino or not currentAssassino.Character then
                pararJogarAssassino()
                return
            end
            
            local targetChar = currentAssassino.Character
            local targetHRP = GetHRP(targetChar)
            local myHRP = GetHRP()
            
            if not targetHRP or not myHRP then
                pararJogarAssassino()
                return
            end
            
            local humanoid = GetHumanoid(targetChar)
            if not humanoid or humanoid.Health <= 0 then
                pararJogarAssassino()
                return
            end
            
            myHRP.RotVelocity = Vector3.new(
                math.random(8000, 9000),
                math.random(8000, 9000),
                math.random(8000, 9000)
            )
            
            local randomAngle = CFrame.Angles(
                math.rad(math.random(-360, 360)),
                math.rad(math.random(-360, 360)),
                math.rad(math.random(-360, 360))
            )
            
            local yOffset = math.random(-6, 6)
            
            myHRP.CFrame = targetHRP.CFrame *
                CFrame.new(
                    math.random(-1, 1),
                    yOffset,
                    math.random(-1, 1)
                ) * randomAngle
            
            myHRP.Velocity = Vector3.new(
                math.random(-500, 500),
                math.random(400, 900),
                math.random(-500, 500)
            )
        end)
    end
})

MM2Tab:Button({
    Title = "Jogar Xerife Longe",
    Callback = function()
        if killSheriffActive then
            pararJogarXerife()
            return
        end
        
        local xerife = encontrarPorPapel("Xerife")
        if not xerife then
            Notify("Nenhum xerife encontrado", "error")
            return
        end
        
        local targetChar = xerife.Character
        local myChar = GetChar()
        
        if not targetChar or not myChar then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local targetHRP = GetHRP(targetChar)
        local myHRP = GetHRP(myChar)
        
        if not targetHRP or not myHRP then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        killSheriffActive = true
        killSheriffPos = myHRP.CFrame
        
        LoopManager:AddFrame("KillSheriffFling", function()
            if not killSheriffActive then
                LoopManager:RemoveFrame("KillSheriffFling")
                return
            end
            
            local currentXerife = encontrarPorPapel("Xerife")
            if not currentXerife or not currentXerife.Character then
                pararJogarXerife()
                return
            end
            
            local targetChar = currentXerife.Character
            local targetHRP = GetHRP(targetChar)
            local myHRP = GetHRP()
            
            if not targetHRP or not myHRP then
                pararJogarXerife()
                return
            end
            
            local humanoid = GetHumanoid(targetChar)
            if not humanoid or humanoid.Health <= 0 then
                pararJogarXerife()
                return
            end
            
            myHRP.RotVelocity = Vector3.new(
                math.random(8000, 9000),
                math.random(8000, 9000),
                math.random(8000, 9000)
            )
            
            local randomAngle = CFrame.Angles(
                math.rad(math.random(-360, 360)),
                math.rad(math.random(-360, 360)),
                math.rad(math.random(-360, 360))
            )
            
            local yOffset = math.random(-6, 6)
            
            myHRP.CFrame = targetHRP.CFrame *
                CFrame.new(
                    math.random(-1, 1),
                    yOffset,
                    math.random(-1, 1)
                ) * randomAngle
            
            myHRP.Velocity = Vector3.new(
                math.random(-500, 500),
                math.random(400, 900),
                math.random(-500, 500)
            )
        end)
    end
})

local flingAllActive = false
local flingAllQueue = {}
local flingAllCurrentTarget = nil
local flingAllConnection = nil
local flingAllCheckConnection = nil
local flingAllLoopRunning = false

local function pararFlingAll()
    if flingAllActive then
        flingAllActive = false
        flingAllQueue = {}
        flingAllCurrentTarget = nil
        if flingAllConnection then
            pcall(function() flingAllConnection:Disconnect() end)
            flingAllConnection = nil
        end
        if flingAllCheckConnection then
            pcall(function() flingAllCheckConnection:Disconnect() end)
            flingAllCheckConnection = nil
        end
        if flingAllLoopRunning then
            LoopManager:RemoveFrame("FlingAll")
            flingAllLoopRunning = false
        end
        Notify("Jogar todos longe desativado", "info")
    end
end

local function flingarJogador(alvo)
    if not alvo or not alvo.Character then
        return false
    end
    
    local targetChar = alvo.Character
    local myChar = GetChar()
    
    if not targetChar or not myChar then
        return false
    end
    
    local targetHRP = GetHRP(targetChar)
    local myHRP = GetHRP(myChar)
    
    if not targetHRP or not myHRP then
        return false
    end
    
    local humanoid = GetHumanoid(targetChar)
    if not humanoid or humanoid.Health <= 0 then
        return false
    end
    
    flingAllCurrentTarget = alvo
    local posicaoInicial = targetHRP.Position
    local flingTimeout = tick() + 5 
    
    if flingAllLoopRunning then
        LoopManager:RemoveFrame("FlingAll")
        flingAllLoopRunning = false
    end
    
    LoopManager:AddFrame("FlingAll", function()
        if not flingAllActive or not flingAllCurrentTarget then
            if flingAllLoopRunning then
                LoopManager:RemoveFrame("FlingAll")
                flingAllLoopRunning = false
            end
            return
        end
        
        if tick() > flingTimeout then
            processarProximoAlvo()
            return
        end
        
        local currentTargetChar = flingAllCurrentTarget.Character
        if not currentTargetChar then
            processarProximoAlvo()
            return
        end
        
        local targetHRP = GetHRP(currentTargetChar)
        local myHRP = GetHRP()
        
        if not targetHRP or not myHRP then
            return
        end
        
        local humanoid = GetHumanoid(currentTargetChar)
        if not humanoid or humanoid.Health <= 0 then
            processarProximoAlvo()
            return
        end
        
        local distanciaAtual = (targetHRP.Position - posicaoInicial).Magnitude
        if distanciaAtual > 300 then
            processarProximoAlvo()
            return
        end
        
        myHRP.RotVelocity = Vector3.new(
            math.random(8000, 9000),
            math.random(8000, 9000),
            math.random(8000, 9000)
        )
        
        local randomAngle = CFrame.Angles(
            math.rad(math.random(-360, 360)),
            math.rad(math.random(-360, 360)),
            math.rad(math.random(-360, 360))
        )
        
        local yOffset = math.random(-6, 6)
        
        myHRP.CFrame = targetHRP.CFrame *
            CFrame.new(
                math.random(-1, 1),
                yOffset,
                math.random(-1, 1)
            ) * randomAngle
        
        myHRP.Velocity = Vector3.new(
            math.random(-500, 500),
            math.random(400, 900),
            math.random(-500, 500)
        )
    end)
    flingAllLoopRunning = true
    
    local function onTargetDied()
        if flingAllActive then
            task.wait(0.5)
            processarProximoAlvo()
        end
    end
    
    if flingAllConnection then
        pcall(function() flingAllConnection:Disconnect() end)
    end
    flingAllConnection = humanoid.Died:Connect(onTargetDied)
    
    return true
end

local function processarProximoAlvo()
    if not flingAllActive then
        return
    end
    
    if flingAllLoopRunning then
        LoopManager:RemoveFrame("FlingAll")
        flingAllLoopRunning = false
    end
    
    if flingAllConnection then
        pcall(function() flingAllConnection:Disconnect() end)
        flingAllConnection = nil
    end
    if flingAllCheckConnection then
        pcall(function() flingAllCheckConnection:Disconnect() end)
        flingAllCheckConnection = nil
    end
    
    if #flingAllQueue == 0 then
        pararFlingAll()
        Notify("Todos os jogadores foram lançados!", "success")
        return
    end
    
    local proximo = table.remove(flingAllQueue, 1)
    flingarJogador(proximo)
end

MM2Tab:Button({
    Title = "Jogar Todos Longe",
    Callback = function()
        if flingAllActive then
            pararFlingAll()
            return
        end
        
        local myChar = GetChar()
        if not myChar then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local myHRP = GetHRP(myChar)
        if not myHRP then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        local myPos = myHRP.Position
        flingAllQueue = {}
        
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player then
                local char = plr.Character
                if char then
                    local hrp = GetHRP(char)
                    if hrp then
                        local distancia = (hrp.Position - myPos).Magnitude
                        if distancia <= 200 then
                            table.insert(flingAllQueue, plr)
                        end
                    end
                end
            end
        end
        
        if #flingAllQueue == 0 then
            Notify("Nenhum jogador próximo encontrado", "error")
            return
        end
        
        flingAllActive = true
        Notify("Lançando " .. #flingAllQueue .. " jogadores...", "info")
        processarProximoAlvo()
    end
})

local function monitorarAssassino()
    local assassino = encontrarPorPapel("Assassino")
    if assassino and assassino.Character then
        local humanoid = GetHumanoid(assassino.Character)
        if humanoid then
            if killAssassinConnection then
                killAssassinConnection:Disconnect()
            end
            killAssassinConnection = humanoid.Died:Connect(function()
                if killAssassinActive then
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
            if killSheriffConnection then
                killSheriffConnection:Disconnect()
            end
            killSheriffConnection = humanoid.Died:Connect(function()
                if killSheriffActive then
                    pararJogarXerife()
                end
            end)
        end
    end
end

local function atualizarMonitoramento()
    monitorarAssassino()
    monitorarXerife()
end

EventManager:Add("PlayerAdded", Players.PlayerAdded:Connect(function()
    atualizarMonitoramento()
end))

for _, plr in pairs(Players:GetPlayers()) do
    EventManager:Add("CharacterAdded_" .. plr.Name, plr.CharacterAdded:Connect(function()
        atualizarMonitoramento()
    end))
end

EventManager:Add("PlayerRemoving", Players.PlayerRemoving:Connect(function(p)
    if killAssassinActive and p == encontrarPorPapel("Assassino") then
        pararJogarAssassino()
    end
    if killSheriffActive and p == encontrarPorPapel("Xerife") then
        pararJogarXerife()
    end
end))

MM2Tab:Section({Title = "Utilidades"})

local autoPickupActive = false
local autoPickupTeleporting = false

local function encontrarArmaNoChao()
    local myChar = GetChar()
    if not myChar then return nil end
    
    local myHRP = GetHRP(myChar)
    if not myHRP then return nil, nil end
    
    local myPos = myHRP.Position
    local armaMaisProxima = nil
    local menorDistancia = 300  
    
    for _, obj in pairs(workspace:GetDescendants()) do
        local isGun = false
        local armaPos = nil
        local nome = obj.Name:lower()
        
        if obj:IsA("BasePart") then
            armaPos = obj.Position
        elseif obj:FindFirstChild("Handle") and obj.Handle:IsA("BasePart") then
            armaPos = obj.Handle.Position
        elseif obj:IsA("Model") and obj.PrimaryPart then
            armaPos = obj.PrimaryPart.Position
        end
        
        if not armaPos then continue end
        
        local distancia = (armaPos - myPos).Magnitude
        if distancia > 300 then continue end 
        
        if obj:IsA("Tool") then
            if nome:find("gun") or nome:find("pistol") or nome:find("revolver") then
                isGun = true
            end
        end
        
        if not isGun and (obj:IsA("Model") or obj:IsA("MeshPart") or obj:IsA("Part")) then
            local hasGreenEffect = false
            for _, child in pairs(obj:GetChildren()) do
                if child:IsA("ParticleEmitter") and child.Enabled then
                    hasGreenEffect = true
                end
                if child:IsA("Highlight") and child.FillColor == Color3.fromRGB(0, 255, 0) then
                    hasGreenEffect = true
                end
            end
            
            if nome:find("gun") or nome:find("pistol") or nome:find("revolver") or nome:find("weapon") then
                isGun = true
            end
            
            if hasGreenEffect then
                isGun = true
            end
        end
        
        if not isGun and obj:FindFirstChild("Handle") then
            local handleName = obj.Handle.Name:lower()
            if handleName:find("gun") or handleName:find("pistol") or handleName:find("revolver") then
                isGun = true
                armaPos = obj.Handle.Position
            end
        end
        
        if isGun and armaPos then
            local parent = obj.Parent
            local isHeld = false
            
            if parent and parent:IsA("Model") and parent:FindFirstChild("Humanoid") then
                local humanoid = parent:FindFirstChild("Humanoid")
                if humanoid and humanoid.Health > 0 then
                    isHeld = true
                end
            end
            
            if not isHeld and parent ~= myChar then
                if armaPos.Y < 30 or (parent and parent.Name == "Workspace") then
                    if distancia < menorDistancia then
                        menorDistancia = distancia
                        armaMaisProxima = obj
                    end
                end
            end
        end
    end
    
    if armaMaisProxima then
        local posFinal = nil
        if armaMaisProxima:IsA("BasePart") then
            posFinal = armaMaisProxima.Position
        elseif armaMaisProxima:FindFirstChild("Handle") then
            posFinal = armaMaisProxima.Handle.Position
        elseif armaMaisProxima.PrimaryPart then
            posFinal = armaMaisProxima.PrimaryPart.Position
        end
        return armaMaisProxima, posFinal
    end
    
    return nil, nil
end

local function teleportarParaArmaEVoltar()
    if autoPickupTeleporting then return end
    
    local myChar = GetChar()
    local myHRP = GetHRP(myChar)
    if not myChar or not myHRP then return end
    
    local arma, armaPos = encontrarArmaNoChao()
    if not arma or not armaPos then return end
    
    autoPickupTeleporting = true
    
    local posOriginal = myHRP.CFrame
    myHRP.CFrame = CFrame.new(armaPos + Vector3.new(0, 3, 0))
    task.wait(0.15)
    
    local success = pcall(function()
        if arma:IsA("Tool") then
            arma.Parent = myChar
        else
            myHRP.CFrame = CFrame.new(armaPos + Vector3.new(0, 1, 0))
            task.wait(0.1)
            
            for _, child in pairs(arma:GetChildren()) do
                if child:IsA("TouchInterest") or child:IsA("ClickDetector") then
                    firetouchinterest(myHRP, child.Parent, 0)
                    firetouchinterest(myHRP, child.Parent, 1)
                end
            end
        end
    end)
    
    if success then
        Notify("Arma detectada!", "success")
    end
    task.wait(0.2)
    myHRP.CFrame = posOriginal
    task.wait(0.5)
    autoPickupTeleporting = false
end

MM2Tab:Toggle({
    Title = "Auto Pegar Pistola (Chão)",
    Default = false,
    Callback = function(Value)
        autoPickupActive = Value
        if Value then
            LoopManager:AddThrottled("AutoPickup", function()
                if autoPickupActive and not autoPickupTeleporting then
                    teleportarParaArmaEVoltar()
                end
            end, 1.5)
        else
            LoopManager:RemoveThrottled("AutoPickup")
        end
    end
})

MM2Tab:Toggle({
    Title = "Auto Farm Moedas",
    Default = false,
    Callback = function(Value)
        AutoFarmToggle(Value)
    end
})

-- ================================ ABA GRÁFICOS ================================

local GraphicsTab = Window:Tab({Title = "Graphics", Icon = "sparkle"})

GraphicsTab:Section({Title = "Time Control"})

GraphicsTab:Slider({
    Title = "Clock Time",
    Step = 1,
    Value = {
        Min = 0,
        Max = 24,
        Default = 12,
    },
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
    Title = "Destravar FPS",
    Default = false,
    Callback = function(Value)
        FPSUnlocked = Value
        pcall(function()
            if Value then
                setfpscap(120)
            else
                setfpscap(60)
            end
        end)
    end
})

-- ================================ ABA ANIMAÇÕES ================================

local AnimationsTab = Window:Tab({Title = "Animations", Icon = "layers"})

AnimationsTab:Section({Title = "Emote Player"})

AnimationsTab:Dropdown({
    Title = "Emotes",
    Values = {
        "Gangnam Style", "Thriller", "Macarena", "Running Man", "The Sprinkler",
        "Carlton Dance", "Moonwalk", "TikTok Dance", "Savage Love", "Say So Dance",
        "Renegade", "Marshmello Dance", "Dab", "Whip", "Nae Nae", "Hit The Quan", "Shoot Dance",
        "Passinho do Jamal", "Passinho do Faraó", "Passinho do Romano", "Passinho do Voltei"
    },
    Default = "Nenhum",
    Callback = function(Value)
        for _, emote in pairs(famousEmotes) do
            if emote.name == Value then
                playEmote(emote.id, emoteSpeed)
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
    Value = {
        Min = 1,
        Max = 10,
        Default = 1,
    },
    Callback = function(Value)
        emoteSpeed = Value
        if currentEmoteTrack then
            currentEmoteTrack:AdjustSpeed(emoteSpeed)
        end
    end
})

AnimationsTab:Section({Title = "Full Packs"})

AnimationsTab:Dropdown({
    Title = "Animation Packs",
    Values = {
        "Cartoony", "Knight", "Toy", "Elder", "Mage",
        "Ninja", "Astronaut", "Zombie", "Pirate", "Vampire",
        "Werewolf", "Levitation", "AdidasCommunity", "AdidasSports", "Ghost",
        "Patrol", "SuperHero", "Sneaky", "Popstar", "Confident", 
        "Stylish", "Robot", "Bubbly", "Princess", "Fashionable",
        "Cutesy", "Bold", "Glitch"
    },
    Default = "Nenhum",
    Callback = function(Value)
        if Value == "Nenhum" then
            selectedAnimationPack = nil
            return
        end
        selectedAnimationPack = Value
        aplicarPack(ANIMATION_PACKS[Value])
    end
})

-- ================================ ABA TELEPORT ================================

local TeleportTab = Window:Tab({Title = "Teleport", Icon = "map-pin"})

TeleportTab:Section({Title = "Principais"})

TeleportTab:Button({
    Title = "Save Pos",
    Callback = function()
        local char = GetChar()
        if not char then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local hrp = GetHRP(char)
        if not hrp then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        savedPosition = hrp.CFrame
        lastPositionTP = hrp.CFrame
        Notify("Posição salva!", "success")
    end
})

TeleportTab:Button({
    Title = "Load Pos",
    Callback = function()
        if not savedPosition then
            Notify("Nenhuma posição salva!", "error")
            return
        end
        
        local char = GetChar()
        if not char then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local hrp = GetHRP(char)
        if not hrp then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        lastPositionTP = hrp.CFrame
        hrp.CFrame = savedPosition
        Notify("Teleportado!", "success")
    end
})

TeleportTab:Button({
    Title = "Return (Last Pos)",
    Callback = function()
        if not lastPositionTP then
            Notify("Nenhuma posição anterior!", "error")
            return
        end
        
        local char = GetChar()
        if not char then
            Notify("Personagem não encontrado", "error")
            return
        end
        
        local hrp = GetHRP(char)
        if not hrp then
            Notify("HumanoidRootPart não encontrado", "error")
            return
        end
        
        hrp.CFrame = lastPositionTP
        Notify("Retornado!", "success")
    end
})
