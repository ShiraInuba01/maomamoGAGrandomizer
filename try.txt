local players = game:GetService("Players")
local collectionService = game:GetService("CollectionService")
local TweenService = game:GetService("TweenService")
local localPlayer = players.LocalPlayer or players:GetPlayers()[1]

local TEAL_BG = Color3.fromRGB(22, 153, 142)
local TEAL_DARK = Color3.fromRGB(0, 108, 87)
local TEAL_HOVER = Color3.fromRGB(0, 140, 110)
local PURPLE = Color3.fromRGB(111, 74, 163)
local PURPLE_HOVER = Color3.fromRGB(140, 100, 200)
local BUTTON_GRAY = Color3.fromRGB(45, 45, 45)
local BUTTON_GRAY_HOVER = Color3.fromRGB(70, 70, 70)
local BUTTON_RED = Color3.fromRGB(200, 62, 62)
local BORDER_COLOR = Color3.fromRGB(0, 80, 70) -- dark teal border
local FONT = Enum.Font.FredokaOne

local eggChances = {
    ["Common Egg"] = {["Dog"] = 33, ["Bunny"] = 33, ["Golden Lab"] = 33},
    ["Uncommon Egg"] = {["Black Bunny"] = 25, ["Chicken"] = 25, ["Cat"] = 25, ["Deer"] = 25},
    ["Rare Egg"] = {["Orange Tabby"] = 33.33, ["Spotted Deer"] = 25, ["Pig"] = 16.67, ["Rooster"] = 16.67, ["Monkey"] = 8.33},
    ["Legendary Egg"] = {["Cow"] = 42.55, ["Silver Monkey"] = 42.55, ["Sea Otter"] = 10.64, ["Turtle"] = 2.13, ["Polar Bear"] = 2.13},
    ["Mythic Egg"] = {["Grey Mouse"] = 37.5, ["Brown Mouse"] = 26.79, ["Squirrel"] = 26.79, ["Red Giant Ant"] = 8.93, ["Red Fox"] = 0},
    ["Bug Egg"] = {["Snail"] = 40, ["Giant Ant"] = 35, ["Caterpillar"] = 25, ["Praying Mantis"] = 7, ["Dragon Fly"] = 3},
    ["Night Egg"] = {["Hedgehog"] = 47, ["Mole"] = 23.5, ["Frog"] = 21.16, ["Echo Frog"] = 8.35, ["Night Owl"] = 0, ["Raccoon"] = 0},
    ["Bee Egg"] = {["Bee"] = 65, ["Honey Bee"] = 20, ["Bear Bee"] = 10, ["Petal Bee"] = 5, ["Queen Bee"] = 0},
    ["Anti Bee Egg"] = {["Wasp"] = 55, ["Tarantula Hawk"] = 31, ["Moth"] = 14, ["Butterfly"] = 0, ["Disco Bee"] = 0},
    ["Common Summer Egg"] = {["Starfish"] = 50, ["Seafull"] = 25, ["Crab"] = 25},
    ["Rare Summer Egg"] = {["Flamingo"] = 30, ["Toucan"] = 25, ["Sea Turtle"] = 20, ["Orangutan"] = 15, ["Seal"] = 10},
    ["Paradise Egg"] = {["Ostrich"] = 43, ["Peacock"] = 33, ["Capybara"] = 24, ["Scarlet Macaw"] = 3, ["Mimic Octopus"] = 1},
    ["Premium Night Egg"] = {["Hedgehog"] = 50, ["Mole"] = 26, ["Frog"] = 14, ["Echo Frog"] = 10}
}

local realESP = {
    ["Common Egg"] = true, ["Uncommon Egg"] = true, ["Rare Egg"] = true,
    ["Common Summer Egg"] = true, ["Rare Summer Egg"] = true
}

local displayedEggs = {}
local autoStopOn = false

local function weightedRandom(options)
    local valid = {}
    for pet, chance in pairs(options) do
        if chance > 0 then table.insert(valid, {pet = pet, chance = chance}) end
    end
    if #valid == 0 then return nil end
    local total = 0
    for _, v in ipairs(valid) do total += v.chance end
    local roll = math.random() * total
    local cumulative = 0
    for _, v in ipairs(valid) do
        cumulative += v.chance
        if roll <= cumulative then return v.pet end
    end
    return valid[1].pet
end

local function getNonRepeatingRandomPet(eggName, lastPet)
    local pool = eggChances[eggName]
    if not pool then return nil end
    local tries, selectedPet = 0, lastPet
    while tries < 5 do
        local pet = weightedRandom(pool)
        if not pet then return nil end
        if pet ~= lastPet or math.random() < 0.3 then
            selectedPet = pet
            break
        end
        tries += 1
    end
    return selectedPet
end

local function createEspGui(object, labelText)
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "FakePetESP"
    billboard.Adornee = object:FindFirstChildWhichIsA("BasePart") or object.PrimaryPart or object
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 2.5, 0)
    billboard.AlwaysOnTop = true

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextStrokeTransparency = 0
    label.TextScaled = true
    label.Font = Enum.Font.SourceSansBold
    label.Text = labelText
    label.Parent = billboard

    billboard.Parent = object
    return billboard
end

local function addESP(egg)
    if egg:GetAttribute("OWNER") ~= localPlayer.Name then return end
    local eggName = egg:GetAttribute("EggName")
    local objectId = egg:GetAttribute("OBJECT_UUID")
    if not eggName or not objectId or displayedEggs[objectId] then return end

    local labelText, firstPet
    if realESP[eggName] then
        labelText = eggName
    else
        firstPet = getNonRepeatingRandomPet(eggName, nil)
        labelText = eggName .. " | " .. (firstPet or "?")
    end

    local espGui = createEspGui(egg, labelText)
    displayedEggs[objectId] = {
        egg = egg,
        gui = espGui,
        label = espGui:FindFirstChild("TextLabel"),
        eggName = eggName,
        lastPet = firstPet
    }
end

local function removeESP(egg)
    local objectId = egg:GetAttribute("OBJECT_UUID")
    if objectId and displayedEggs[objectId] then
        displayedEggs[objectId].gui:Destroy()
        displayedEggs[objectId] = nil
    end
end

for _, egg in collectionService:GetTagged("PetEggServer") do
    addESP(egg)
end

collectionService:GetInstanceAddedSignal("PetEggServer"):Connect(addESP)
collectionService:GetInstanceRemovedSignal("PetEggServer"):Connect(removeESP)

local gui = Instance.new("ScreenGui")
gui.Name = "RandomizerStyledGUI"
gui.ResetOnSpawn = false
gui.Parent = localPlayer:WaitForChild("PlayerGui")

-- Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 260, 0, 120)
mainFrame.Position = UDim2.new(0.5, -130, 0.5, -60)
mainFrame.BackgroundColor3 = TEAL_BG
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = BORDER_COLOR
mainFrame.Parent = gui
mainFrame.Active = true
mainFrame.Draggable = true
local frameCorner = Instance.new("UICorner", mainFrame)
frameCorner.CornerRadius = UDim.new(0, 8)

-- Top Bar
local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 24)
topBar.BackgroundColor3 = TEAL_BG
topBar.BorderSizePixel = 1
topBar.BorderColor3 = BORDER_COLOR
topBar.Parent = mainFrame
local topBarCorner = Instance.new("UICorner", topBar)
topBarCorner.CornerRadius = UDim.new(0, 8)

-- Info Button (left)
local infoBtn = Instance.new("TextButton")
infoBtn.Size = UDim2.new(0, 20, 0, 20)
infoBtn.Position = UDim2.new(0, 2, 0.5, -10)
infoBtn.BackgroundColor3 = BUTTON_GRAY
infoBtn.Text = "?"
infoBtn.Font = FONT
infoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
infoBtn.TextScaled = true
infoBtn.TextStrokeTransparency = 0.7
infoBtn.Parent = topBar
infoBtn.ZIndex = 2
infoBtn.AutoButtonColor = false
infoBtn.BackgroundTransparency = 0
infoBtn.BorderSizePixel = 1
infoBtn.BorderColor3 = BORDER_COLOR
local infoCorner = Instance.new("UICorner", infoBtn)
infoCorner.CornerRadius = UDim.new(1, 0)
infoBtn.MouseEnter:Connect(function()
    infoBtn.BackgroundColor3 = BUTTON_GRAY_HOVER
end)
infoBtn.MouseLeave:Connect(function()
    infoBtn.BackgroundColor3 = BUTTON_GRAY
end)

-- Title Label (centered)
local topLabel = Instance.new("TextLabel")
topLabel.Size = UDim2.new(1, -70, 1, 0)
topLabel.Position = UDim2.new(0, 28, 0, 0)
topLabel.BackgroundTransparency = 1
topLabel.Text = "MaoMao’s Pet Randomizer"
topLabel.Font = FONT
topLabel.TextColor3 = Color3.new(1, 1, 1)
topLabel.TextStrokeTransparency = 0.7
topLabel.TextScaled = true
topLabel.TextXAlignment = Enum.TextXAlignment.Left
topLabel.ZIndex = 1
topLabel.Parent = topBar

-- Close Button (right)
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 20, 0, 20)
closeBtn.Position = UDim2.new(1, -22, 0.5, -10)
closeBtn.BackgroundColor3 = BUTTON_GRAY
closeBtn.Text = "X"
closeBtn.Font = FONT
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
closeBtn.TextScaled = true
closeBtn.TextStrokeTransparency = 0.7
closeBtn.Parent = topBar
closeBtn.ZIndex = 2
closeBtn.AutoButtonColor = false
closeBtn.BackgroundTransparency = 0
closeBtn.BorderSizePixel = 1
closeBtn.BorderColor3 = BORDER_COLOR
local closeCorner = Instance.new("UICorner", closeBtn)
closeCorner.CornerRadius = UDim.new(1, 0)
closeBtn.MouseEnter:Connect(function()
    closeBtn.BackgroundColor3 = BUTTON_RED
end)
closeBtn.MouseLeave:Connect(function()
    closeBtn.BackgroundColor3 = BUTTON_GRAY
end)
closeBtn.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- Content Frame
local contentFrame = Instance.new("Frame")
contentFrame.Name = "ContentFrame"
contentFrame.Size = UDim2.new(1, -16, 1, -36)
contentFrame.Position = UDim2.new(0, 8, 0, 28)
contentFrame.BackgroundTransparency = 1
contentFrame.ZIndex = 2
contentFrame.Parent = mainFrame

-- Styled Buttons
local function makeStyledButton(text, yPos, color, hover, onHover, onUnhover)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 32)
    btn.Position = UDim2.new(0, 0, 0, yPos)
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.Font = FONT
    btn.TextColor3 = Color3.new(1,1,1)
    btn.TextScaled = true
    btn.TextStrokeTransparency = 0.7
    btn.ZIndex = 2
    btn.Parent = contentFrame
    btn.AutoButtonColor = false
    btn.BorderSizePixel = 1
    btn.BorderColor3 = BORDER_COLOR
    local btnCorner = Instance.new("UICorner", btn)
    btnCorner.CornerRadius = UDim.new(0.5, 0)
    btn.MouseEnter:Connect(function()
        if onHover then onHover(btn) else btn.BackgroundColor3 = hover end
    end)
    btn.MouseLeave:Connect(function()
        if onUnhover then onUnhover(btn) else btn.BackgroundColor3 = color end
    end)
    return btn
end

local function updateStopBtnColors(btn)
    if autoStopOn then
        btn.BackgroundColor3 = TEAL_DARK
        btn.Text = "[A] Auto Stop: On"
        btn.TextColor3 = Color3.new(1,1,1)
    else
        btn.BackgroundColor3 = TEAL_DARK
        btn.Text = "[A] Auto Stop: Off"
        btn.TextColor3 = Color3.new(1,1,1)
    end
end

local stopBtn = makeStyledButton(
    "[A] Auto Stop: Off",
    0,
    TEAL_DARK,
    TEAL_HOVER
)
updateStopBtnColors(stopBtn)

local rerollBtn = makeStyledButton(
    "[B] Re-roll Pet",
    40,
    PURPLE,
    PURPLE_HOVER
)

stopBtn.MouseButton1Click:Connect(function()
    autoStopOn = not autoStopOn
    updateStopBtnColors(stopBtn)
end)
rerollBtn.MouseButton1Click:Connect(function()
    local stopPets = {
        ["Raccoon"] = true, ["Dragon Fly"] = true, ["Queen Bee"] = true,
        ["Red Fox"] = true, ["Disco Bee"] = true, ["Butterfly"] = true
    }
    for objectId, data in pairs(displayedEggs) do
        local pet = getNonRepeatingRandomPet(data.eggName, data.lastPet)
        if pet and data.label then
            data.label.Text = data.eggName .. " | " .. pet
            data.lastPet = pet
            if autoStopOn and stopPets[pet] then
                autoStopOn = false
                updateStopBtnColors(stopBtn)
                break -- stop rerolling further eggs
            end
        end
    end
end)

local camera = workspace.CurrentCamera
local originalFOV
local zoomFOV = 60
local tweenTime = 0.4
local currentTween

infoBtn.MouseButton1Click:Connect(function()
    if gui:FindFirstChild("InfoModal") then
        return
    end

    local blur = Instance.new("BlurEffect")
    blur.Size = 16
    blur.Name = "ModalBlur"
    blur.Parent = game:GetService("Lighting")

    if camera then
        originalFOV = camera.FieldOfView
        if currentTween then currentTween:Cancel() end
        currentTween = TweenService:Create(camera, TweenInfo.new(tweenTime, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            FieldOfView = zoomFOV
        })
        currentTween:Play()
    end

    local modal = Instance.new("Frame")
    modal.Name = "InfoModal"
    modal.Size = UDim2.new(0, 180, 0, 70)
    modal.Position = UDim2.new(0.5, -90, 0.5, -35)
    modal.BackgroundColor3 = TEAL_BG
    modal.Active = true
    modal.ZIndex = 30
    modal.BorderSizePixel = 2
    modal.BorderColor3 = BORDER_COLOR
    modal.Parent = gui
    local modalCorner = Instance.new("UICorner", modal)
    modalCorner.CornerRadius = UDim.new(0, 8)

    local textTile = Instance.new("Frame")
    textTile.Size = UDim2.new(1, 0, 0, 18)
    textTile.Position = UDim2.new(0, 0, 0, 0)
    textTile.BackgroundColor3 = TEAL_DARK
    textTile.ZIndex = 30
    textTile.BorderSizePixel = 1
    textTile.BorderColor3 = BORDER_COLOR
    textTile.Parent = modal
    local textTileCorner = Instance.new("UICorner", textTile)
    textTileCorner.CornerRadius = UDim.new(0, 8)

    local textTileLabel = Instance.new("TextLabel")
    textTileLabel.Size = UDim2.new(1, -20, 1, 0)
    textTileLabel.Position = UDim2.new(0, 8, 0, 0)
    textTileLabel.BackgroundTransparency = 1
    textTileLabel.Text = "Info"
    textTileLabel.TextColor3 = Color3.fromRGB(255,255,255)
    textTileLabel.Font = FONT
    textTileLabel.TextScaled = true
    textTileLabel.ZIndex = 31
    textTileLabel.TextStrokeTransparency = 0.7
    textTileLabel.Parent = textTile

    local closeBtn2 = Instance.new("TextButton")
    closeBtn2.Size = UDim2.new(0, 16, 0, 16)
    closeBtn2.Position = UDim2.new(1, -18, 0, 1)
    closeBtn2.BackgroundColor3 = BUTTON_GRAY
    closeBtn2.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn2.Text = "✖"
    closeBtn2.TextScaled = true
    closeBtn2.Font = FONT
    closeBtn2.ZIndex = 32
    closeBtn2.AutoButtonColor = false
    closeBtn2.BorderSizePixel = 1
    closeBtn2.BorderColor3 = BORDER_COLOR
    closeBtn2.Parent = textTile
    local closeCorner2 = Instance.new("UICorner", closeBtn2)
    closeCorner2.CornerRadius = UDim.new(1, 0)
    closeBtn2.MouseEnter:Connect(function()
        closeBtn2.BackgroundColor3 = BUTTON_RED
    end)
    closeBtn2.MouseLeave:Connect(function()
        closeBtn2.BackgroundColor3 = BUTTON_GRAY
    end)
    closeBtn2.MouseButton1Click:Connect(function()
        if blur then blur:Destroy() end
        if modal then modal:Destroy() end
        if camera and originalFOV then
            if currentTween then currentTween:Cancel() end
            currentTween = TweenService:Create(camera, TweenInfo.new(tweenTime, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                FieldOfView = originalFOV
            })
            currentTween:Play()
        end
    end)

    local infoBox = Instance.new("Frame")
    infoBox.Size = UDim2.new(1, -10, 1, -21)
    infoBox.Position = UDim2.new(0, 5, 0, 16)
    infoBox.BackgroundColor3 = TEAL_BG
    infoBox.BackgroundTransparency = 0
    infoBox.ZIndex = 30
    infoBox.BorderSizePixel = 1
    infoBox.BorderColor3 = BORDER_COLOR
    infoBox.Parent = modal

    local infoBoxCorner = Instance.new("UICorner", infoBox)
    infoBoxCorner.CornerRadius = UDim.new(0, 7)

    local infoLabel = Instance.new("TextLabel")
    infoLabel.Size = UDim2.new(1, 0, 1, 0)
    infoLabel.Position = UDim2.new(0, 0, 0, 0)
    infoLabel.BackgroundTransparency = 1
    infoLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    infoLabel.Text = "Auto Stop when found:\nRaccoon, Dragonfly, Queen Bee, Red Fox, Disco Bee, Butterfly."
    infoLabel.TextWrapped = true
    infoLabel.Font = FONT
    infoLabel.TextScaled = true
    infoLabel.ZIndex = 31
    infoLabel.TextStrokeTransparency = 0.7
    infoLabel.Parent = infoBox
end)