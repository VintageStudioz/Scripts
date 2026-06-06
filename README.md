-- =============================================================================
-- Vintage Studios UI Framework (Unified Base Template)
-- Description: Multi-page UI system handling 12 distinct views,
-- cross-platform input wrapping (PC/Mobile), and layout structures.
-- =============================================================================

-- Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local ContextActionService = game:GetService("ContextActionService")
local TweenService = game:GetService("TweenService")

-- Ensure PlayerGui is loaded
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- 1. ScreenGui Initialization
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VintageStudiosInterface"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

-- 2. Loading Notification Banner
local LoadingBanner = Instance.new("TextLabel")
LoadingBanner.Size = UDim2.new(0, 320, 0, 60)
LoadingBanner.Position = UDim2.new(0.5, -160, 0.4, -30)
LoadingBanner.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
LoadingBanner.TextColor3 = Color3.fromRGB(240, 240, 240)
LoadingBanner.Text = "Vintage Studios is opening..."
LoadingBanner.Font = Enum.Font.GothamMedium
LoadingBanner.TextSize = 18
LoadingBanner.BorderSizePixel = 0
LoadingBanner.Parent = ScreenGui

local BannerCorner = Instance.new("UICorner")
BannerCorner.CornerRadius = UDim.new(0, 8)
BannerCorner.Parent = LoadingBanner

-- Hold layout display for 2 seconds as requested, then fade out
task.wait(2)
local fadeTween = TweenService:Create(LoadingBanner, TweenInfo.new(0.5), {TextTransparency = 1, BackgroundTransparency = 1})
fadeTween:Play()
fadeTween.Completed:Connect(function()
    LoadingBanner:Destroy()
end)

-- 3. Main Window Panel
local MainPanel = Instance.new("Frame")
MainPanel.Size = UDim2.new(0, 600, 0, 400)
MainPanel.Position = UDim2.new(0.5, -300, 0.5, -200)
MainPanel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainPanel.BorderSizePixel = 0
MainPanel.Active = true
MainPanel.Visible = true
MainPanel.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainPanel

-- 4. Header & Profile Configuration Area
local ProfileImage = Instance.new("ImageLabel")
ProfileImage.Size = UDim2.new(0, 45, 0, 45)
ProfileImage.Position = UDim2.new(0, 15, 0, 15)
ProfileImage.BackgroundTransparency = 1
-- NOTE: Local PC paths (file:///C:/...) are inaccessible to the online engine.
-- Upload your file via the Roblox Creator Dashboard and update this asset ID string.
ProfileImage.Image = "rbxassetid://0" 
ProfileImage.Parent = MainPanel

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(0, 200, 0, 45)
TitleLabel.Position = UDim2.new(0, 70, 0, 15)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "Vintage Studios"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 18
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = MainPanel

-- 5. Scrollable Sidebar Setup
local NavigationFrame = Instance.new("ScrollingFrame")
NavigationFrame.Size = UDim2.new(0, 150, 1, -80)
NavigationFrame.Position = UDim2.new(0, 15, 0, 70)
NavigationFrame.BackgroundTransparency = 1
NavigationFrame.CanvasSize = UDim2.new(0, 0, 0, 520) -- Accommodates 12 navigation entries
NavigationFrame.ScrollBarThickness = 4
NavigationFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 100)
NavigationFrame.Parent = MainPanel

local NavLayout = Instance.new("UIListLayout")
NavLayout.Padding = UDim.new(0, 6)
NavLayout.SortOrder = Enum.SortOrder.LayoutOrder
NavLayout.Parent = NavigationFrame

-- 6. Central Content Viewport Window
local ContentCanvas = Instance.new("Frame")
ContentCanvas.Size = UDim2.new(1, -195, 1, -80)
ContentCanvas.Position = UDim2.new(0, 180, 0, 70)
ContentCanvas.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ContentCanvas.Parent = MainPanel

local CanvasCorner = Instance.new("UICorner")
CanvasCorner.CornerRadius = UDim.new(0, 8)
CanvasCorner.Parent = ContentCanvas

-- =============================================================================
-- INTERACTION MECHANICS (Dragging Framework)
-- =============================================================================
local isDragging = false
local dragInputSource, inputStartPosition, panelStartPosition

local function updatePanelPosition(input)
    local positionDelta = input.Position - inputStartPosition
    MainPanel.Position = UDim2.new(
        panelStartPosition.X.Scale, 
        panelStartPosition.X.Offset + positionDelta.X, 
        panelStartPosition.Y.Scale, 
        panelStartPosition.Y.Offset + positionDelta.Y
    )
end

MainPanel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        inputStartPosition = input.Position
        panelStartPosition = MainPanel.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                isDragging = false
            end
        end)
    end
end)

MainPanel.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInputSource = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInputSource and isDragging then
        updatePanelPosition(input)
    end
end)

-- =============================================================================
-- PAGE SYSTEM ALLOCATION & CREATION
-- =============================================================================
local panelPages = {}
local activePageFrame = nil

local function ConstructPanelPage(name)
    local PageFrame = Instance.new("Frame")
    PageFrame.Size = UDim2.new(1, 0, 1, 0)
    PageFrame.BackgroundTransparency = 1
    PageFrame.Visible = false
    PageFrame.Parent = ContentCanvas
    
    panelPages[name] = PageFrame
    
    -- Instantiate Sidebar Switch Button
    local SelectButton = Instance.new("TextButton")
    SelectButton.Size = UDim2.new(1, -8, 0, 32)
    SelectButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    SelectButton.TextColor3 = Color3.fromRGB(230, 230, 230)
    SelectButton.Text = name
    SelectButton.Font = Enum.Font.Gotham
    SelectButton.TextSize = 12
    SelectButton.Parent = NavigationFrame
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 4)
    ButtonCorner.Parent = SelectButton
    
    SelectButton.MouseButton1Click:Connect(function()
        if activePageFrame then
            activePageFrame.Visible = false
        end
        PageFrame.Visible = true
        activePageFrame = PageFrame
    end)
    
    return PageFrame
end

-- Initialize 12 Distinct Layout View Containers
local AimbotPage      = ConstructPanelPage("Aimbot")
local ESPPage         = ConstructPanelPage("ESP Systems")
local WallhackPage    = ConstructPanelPage("Wallhacks")
local FOVPage         = ConstructPanelPage("Field of View")
local AutoFarmPage    = ConstructPanelPage("Autofarm Engine")
local AutoShootPage   = ConstructPanelPage("Auto Shoot")
local AntiCheatPage   = ConstructPanelPage("Anti-Cheat Module")
local WindowSizePage  = ConstructPanelPage("Window Controls")
local RapidFirePage   = ConstructPanelPage("Rapid Fire Settings")
local AutoCratePage   = ConstructPanelPage("Crate System")
local VisibilityPage  = ConstructPanelPage("Visibility Logic")
local SilentAimPage   = ConstructPanelPage("Silent Targeting")

-- Reveal Initial Default Frame View
panelPages["Aimbot"].Visible = true
activePageFrame = panelPages["Aimbot"]

-- Add UI Placeholders to track content per page visually
for pageName, frame in pairs(panelPages) do
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 40)
    label.Position = UDim2.new(0, 0, 0, 10)
    label.BackgroundTransparency = 1
    label.Text = pageName .. " Configuration Interface"
    label.TextColor3 = Color3.fromRGB(180, 180, 180)
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 14
    label.Parent = frame
end

-- =============================================================================
-- FIELD OF VIEW SLIDER CONSTRAINTS (50 to 1000, Default 200)
-- =============================================================================
local SliderBackground = Instance.new("Frame")
SliderBackground.Size = UDim2.new(0, 260, 0, 8)
SliderBackground.Position = UDim2.new(0.5, -130, 0.5, -4)
SliderBackground.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
SliderBackground.Parent = FOVPage

local SliderKnob = Instance.new("TextButton")
SliderKnob.Size = UDim2.new(0, 18, 0, 18)
-- Default value 200 sits at approximately 15.7% scale distance along track mapping
SliderKnob.Position = UDim2.new(0.157, -9, 0.5, -9) 
SliderKnob.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
SliderKnob.Text = ""
SliderKnob.Parent = SliderBackground

local SliderKnobCorner = Instance.new("UICorner")
SliderKnobCorner.CornerRadius = UDim.new(1, 0)
SliderKnobCorner.Parent = SliderKnob

local ValueIndicator = Instance.new("TextLabel")
ValueIndicator.Size = UDim2.new(0, 200, 0, 25)
ValueIndicator.Position = UDim2.new(0.5, -100, 0.5, -40)
ValueIndicator.BackgroundTransparency = 1
ValueIndicator.TextColor3 = Color3.fromRGB(255, 255, 255)
ValueIndicator.Font = Enum.Font.GothamBold
ValueIndicator.TextSize = 14
ValueIndicator.Text = "FOV Radius: 200"
ValueIndicator.Parent = FOVPage

local slidingActive = false

SliderKnob.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        slidingActive = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        slidingActive = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if slidingActive and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local relativeX = math.clamp(input.Position.X - SliderBackground.AbsolutePosition.X, 0, SliderBackground.AbsoluteSize.X)
        local percentage = relativeX / SliderBackground.AbsoluteSize.X
        
        -- Interpolate mapping scale smoothly between limits 50 and 1000
        local calculatedValue = math.round(50 + (percentage * (1000 - 50)))
        
        SliderKnob.Position = UDim2.new(percentage, -9, 0.5, -9)
        ValueIndicator.Text = "FOV Radius: " .. tostring(calculatedValue)
    end
end)

-- =============================================================================
-- PROXIMITY ALERT PLACEHOLDER LOOPS
-- =============================================================================
local AlertLabel = Instance.new("TextLabel")
AlertLabel.Size = UDim2.new(0, 200, 0, 30)
AlertLabel.Position = UDim2.new(0.5, -100, 0, 20)
AlertLabel.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
AlertLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
AlertLabel.Font = Enum.Font.GothamBold
AlertLabel.TextSize = 14
AlertLabel.Text = "ALERT: Player Nearby"
AlertLabel.Visible = false
AlertLabel.Parent = ScreenGui

local AlertCorner = Instance.new("UICorner")
AlertCorner.CornerRadius = UDim.new(0, 6)
AlertCorner.Parent = AlertLabel

-- Safe background parsing loop processing relative distances
task.spawn(function()
    while task.wait(1) do
        local character = LocalPlayer.Character
        local rootPart = character and character:FindFirstChild("HumanoidRootPart")
        local structuralProximityAlert = false
        
        if rootPart then
            for _, otherPlayer in ipairs(Players:GetPlayers()) do
                if otherPlayer ~= LocalPlayer and otherPlayer.Character then
                    local enemyRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if enemyRoot then
                        -- Check distance threshold (e.g., within 30 studs)
                        local distance = (rootPart.Position - enemyRoot.Position).Magnitude
                        if distance < 30 then
                            structuralProximityAlert = true
                            break
                        end
                    end
                end
            end
        end
        AlertLabel.Visible = structuralProximityAlert
    end
end)

-- =============================================================================
-- KEYBOARD BINDINGS & DYNAMIC MOBILE ACTIONS
-- =============================================================================
local function handleContextAction(actionName, inputState, inputObj)
    if inputState == Enum.UserInputState.Begin then
        print("Action Execution Call: " .. tostring(actionName))
        
        -- Native execution handling for interface minimization
        if actionName == "MinimizeToggle" then
            MainPanel.Visible = not MainPanel.Visible
        end
    end
end

-- Wrapper handles registration and touch platform conversions safely
local function SetupActionBind(actionId, defaultKey, buttonLabel)
    -- Third boolean parameter generates contextual interaction items automatically on mobile
    ContextActionService:BindAction(actionId, handleContextAction, true, defaultKey)
    ContextActionService:SetTitle(actionId, buttonLabel)
    
    local actionButton = ContextActionService:GetButton(actionId)
    if actionButton then
        actionButton.Size = UDim2.new(0, 60, 0, 60)
        actionButton.Active = true -- Enables touch dragging positions cleanly natively
    end
end

-- Map configurations across platform devices uniformly
SetupActionBind("AimbotBind",       Enum.KeyCode.A,   "Lock Head")
SetupActionBind("ESPBind",          Enum.KeyCode.E,   "Toggle ESP")
SetupActionBind("WallhackBind",     Enum.KeyCode.W,   "Wallhack")
SetupActionBind("AutoShootBind",    Enum.KeyCode.J,   "AutoShoot")
SetupActionBind("AntiCheatBind",    Enum.KeyCode.K,   "AntiCheat")
SetupActionBind("MinimizeToggle",   Enum.KeyCode.Tab, "Minimize")
SetupActionBind("RapidFireBind",    Enum.KeyCode.N,   "RapidFire")
SetupActionBind("AutoCrateBind",    Enum.KeyCode.M,   "OpenCrate")
SetupActionBind("VisibilityBind",   Enum.KeyCode.X,   "VisibleOnly")
SetupActionBind("CrosshairBind",    Enum.KeyCode.O,   "CrossOn")
SetupActionBind("PredictionBind",   Enum.KeyCode.L,   "Predict")
SetupActionBind("FOVToggleBind",    Enum.KeyCode.H,   "FOV UI")
SetupActionBind("SilentAimBind",    Enum.KeyCode.T,   "SilentAim")
