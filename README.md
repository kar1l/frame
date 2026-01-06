

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- Loading Screen
local function showLoadingScreen()
	local pg = LocalPlayer:WaitForChild("PlayerGui")
	
	local loadingGui = Instance.new("ScreenGui")
	loadingGui.Name = "HolyGrailLoading"
	loadingGui.ResetOnSpawn = false
	loadingGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	loadingGui.Parent = pg
	
	-- Small centered loading box
	local bg = Instance.new("Frame")
	bg.Size = UDim2.new(0, 280, 0, 120)
	bg.Position = UDim2.new(0.5, -140, 0.5, -60)
	bg.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
	bg.BorderSizePixel = 0
	bg.Parent = loadingGui
	
	Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 12)
	local bgStroke = Instance.new("UIStroke", bg)
	bgStroke.Color = Color3.fromRGB(45, 45, 60)
	bgStroke.Thickness = 1
	
	local bgGradient = Instance.new("UIGradient")
	bgGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(22, 22, 30)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(16, 16, 22))
	}
	bgGradient.Rotation = 90
	bgGradient.Parent = bg
	
	local titleLabel = Instance.new("TextLabel")
	titleLabel.Size = UDim2.new(1, 0, 0, 28)
	titleLabel.Position = UDim2.new(0, 0, 0, 16)
	titleLabel.BackgroundTransparency = 1
	titleLabel.Text = "Holy Grail"
	titleLabel.TextColor3 = Color3.fromRGB(25, 60, 180)
	titleLabel.Font = Enum.Font.GothamBold
	titleLabel.TextSize = 22
	titleLabel.Parent = bg
	
	local subtitleLabel = Instance.new("TextLabel")
	subtitleLabel.Size = UDim2.new(1, 0, 0, 16)
	subtitleLabel.Position = UDim2.new(0, 0, 0, 42)
	subtitleLabel.BackgroundTransparency = 1
	subtitleLabel.Text = "Holy Grail Frame"
	subtitleLabel.TextColor3 = Color3.fromRGB(140, 140, 160)
	subtitleLabel.Font = Enum.Font.GothamMedium
	subtitleLabel.TextSize = 11
	subtitleLabel.Parent = bg
	
	-- Loading bar background
	local barBg = Instance.new("Frame")
	barBg.Size = UDim2.new(0.85, 0, 0, 4)
	barBg.Position = UDim2.new(0.075, 0, 0, 72)
	barBg.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
	barBg.BorderSizePixel = 0
	barBg.Parent = bg
	Instance.new("UICorner", barBg).CornerRadius = UDim.new(0, 2)
	
	-- Loading bar fill
	local barFill = Instance.new("Frame")
	barFill.Size = UDim2.new(0, 0, 1, 0)
	barFill.BackgroundColor3 = Color3.fromRGB(25, 60, 180)
	barFill.BorderSizePixel = 0
	barFill.Parent = barBg
	Instance.new("UICorner", barFill).CornerRadius = UDim.new(0, 2)
	
	local statusLabel = Instance.new("TextLabel")
	statusLabel.Size = UDim2.new(1, 0, 0, 14)
	statusLabel.Position = UDim2.new(0, 0, 0, 88)
	statusLabel.BackgroundTransparency = 1
	statusLabel.Text = "Initializing..."
	statusLabel.TextColor3 = Color3.fromRGB(100, 100, 120)
	statusLabel.Font = Enum.Font.GothamMedium
	statusLabel.TextSize = 10
	statusLabel.Parent = bg
	
	-- Animate loading
	local loadingSteps = {
		{progress = 0.15, text = "Loading services..."},
		{progress = 0.30, text = "Setting up cache..."},
		{progress = 0.50, text = "Initializing avatar system..."},
		{progress = 0.70, text = "Preparing UI..."},
		{progress = 0.90, text = "Finalizing..."},
		{progress = 1.0, text = "Ready!"}
	}
	
	for _, step in ipairs(loadingSteps) do
		local tween = TweenService:Create(barFill, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {
			Size = UDim2.new(step.progress, 0, 1, 0)
		})
		tween:Play()
		statusLabel.Text = step.text
		task.wait(0.2)
	end
	
	task.wait(0.3)
	
	-- Fade out
	local fadeOut = TweenService:Create(bg, TweenInfo.new(0.5, Enum.EasingStyle.Quart), {
		BackgroundTransparency = 1
	})
	local titleFade = TweenService:Create(titleLabel, TweenInfo.new(0.5), {TextTransparency = 1})
	local subtitleFade = TweenService:Create(subtitleLabel, TweenInfo.new(0.5), {TextTransparency = 1})
	local statusFade = TweenService:Create(statusLabel, TweenInfo.new(0.5), {TextTransparency = 1})
	local barBgFade = TweenService:Create(barBg, TweenInfo.new(0.5), {BackgroundTransparency = 1})
	local barFillFade = TweenService:Create(barFill, TweenInfo.new(0.5), {BackgroundTransparency = 1})
	
	fadeOut:Play()
	titleFade:Play()
	subtitleFade:Play()
	statusFade:Play()
	barBgFade:Play()
	barFillFade:Play()
	
	task.wait(0.5)
	loadingGui:Destroy()
end

-- Show loading screen
showLoadingScreen()

-- Store active monitoring connections per player
local activeConnections = {}

-- Store source avatar item data for examine window interception
local sourceAvatarItems = {}

-- Store original player data for reverting
local originalPlayerData = {}

-- Respawn handling toggle (can be disabled for problematic games)
local autoRespawnEnabled = true

-- Avatar application safety mode - skips potentially problematic operations
local safeModeEnabled = false

-- Extra delay before avatar application
local extraDelayEnabled = false

-- Caching system to reduce API calls
local userInfoCache = {}
local avatarModelCache = {}
local humanoidDescCache = {}
local CACHE_EXPIRY = 300

local function getCacheKey(userId, cacheType)
	return cacheType .. "_" .. tostring(userId)
end

local function isCacheValid(cacheEntry)
	if not cacheEntry then return false end
	return (tick() - cacheEntry.timestamp) < CACHE_EXPIRY
end

local function getCachedUserInfo(userId)
	local key = getCacheKey(userId, "userinfo")
	local cached = userInfoCache[key]
	if cached and isCacheValid(cached) then
		return cached.data.username, cached.data.displayName
	end
	return nil, nil
end

local function setCachedUserInfo(userId, username, displayName)
	local key = getCacheKey(userId, "userinfo")
	userInfoCache[key] = {
		data = {username = username, displayName = displayName},
		timestamp = tick()
	}
end

local function getCachedAvatarModel(userId)
	local key = getCacheKey(userId, "avatarmodel")
	local cached = avatarModelCache[key]
	if cached and isCacheValid(cached) then
		return cached.data
	end
	return nil
end

local function setCachedAvatarModel(userId, model)
	local key = getCacheKey(userId, "avatarmodel")
	avatarModelCache[key] = {
		data = model,
		timestamp = tick()
	}
end

local function getCachedHumanoidDescription(userId)
	local key = getCacheKey(userId, "humdesc")
	local cached = humanoidDescCache[key]
	if cached and isCacheValid(cached) then
		return cached.data
	end
	return nil
end

local function setCachedHumanoidDescription(userId, humDesc)
	local key = getCacheKey(userId, "humdesc")
	humanoidDescCache[key] = {
		data = humDesc,
		timestamp = tick()
	}
end

-- Memory cleanup functions
local function cleanupPlayerConnections(userId)
	if activeConnections[userId] then
		for _, connection in ipairs(activeConnections[userId]) do
			if connection and typeof(connection) == "RBXScriptConnection" and connection.Connected then
				pcall(function() connection:Disconnect() end)
			elseif connection and typeof(connection) == "table" and connection.Disconnect then
				pcall(function() connection:Disconnect() end)
			end
		end
		activeConnections[userId] = nil
	end
end

local function cleanupExpiredCache()
	local currentTime = tick()
	local toRemove = {}

	-- Clean expired user info cache
	for key, entry in pairs(userInfoCache) do
		if not isCacheValid(entry) then
			table.insert(toRemove, key)
		end
	end
	for _, key in ipairs(toRemove) do
		userInfoCache[key] = nil
	end

	-- Clean expired avatar model cache
	toRemove = {}
	for key, entry in pairs(avatarModelCache) do
		if not isCacheValid(entry) then
			table.insert(toRemove, key)
		end
	end
	for _, key in ipairs(toRemove) do
		avatarModelCache[key] = nil
	end

	-- Clean expired humanoid description cache
	toRemove = {}
	for key, entry in pairs(humanoidDescCache) do
		if not isCacheValid(entry) then
			table.insert(toRemove, key)
		end
	end
	for _, key in ipairs(toRemove) do
		humanoidDescCache[key] = nil
	end
end

-- Simple logging system
local LOG_LEVEL = {
	ERROR = 1,
	WARN = 2,
	INFO = 3,
	DEBUG = 4
}

local currentLogLevel = LOG_LEVEL.INFO

-- ============================================
-- STAT SPOOFING SYSTEM
-- ============================================

-- Storage for original stat values (for reverting)
local originalStats = {}

-- Format large numbers for display (0-999,999 as is, above that as 1M, 1.5M, etc.)
local function formatStatNumber(num)
	if type(num) ~= "number" then
		num = tonumber(num) or 0
	end
	
	if num < 1000000 then
		-- For numbers under 1M, use comma formatting
		local formatted = tostring(math.floor(num))
		local k
		while true do
			formatted, k = formatted:gsub("^(-?%d+)(%d%d%d)", "%1,%2")
			if k == 0 then break end
		end
		return formatted
	else
		-- For numbers 1M and above, show as M format
		local millions = num / 1000000
		if millions >= 10 then
			return string.format("%.0fM", millions)
		else
			-- Show one decimal for numbers like 1.5M
			local formatted = string.format("%.1fM", millions)
			-- Remove .0 if it's a whole number
			return formatted:gsub("%.0M", "M")
		end
	end
end

-- Parse stat input (handles raw numbers, formatted numbers, and M/K suffixes)
local function parseStatInput(input)
	if not input or input == "" then return nil end
	
	-- Remove commas and spaces
	input = tostring(input):gsub(",", ""):gsub("%s", "")
	
	-- Handle M suffix (millions)
	local mMatch = input:match("^([%d%.]+)[Mm]$")
	if mMatch then
		return math.floor(tonumber(mMatch) * 1000000)
	end
	
	-- Handle K suffix (thousands)
	local kMatch = input:match("^([%d%.]+)[Kk]$")
	if kMatch then
		return math.floor(tonumber(kMatch) * 1000)
	end
	
	-- Raw number
	return tonumber(input)
end

-- Spoof a player's leaderstats
local function spoofLeaderstats(player, statName, newValue)
	if not player then return false, "Player not found" end
	
	local leaderstats = player:FindFirstChild("leaderstats")
	if not leaderstats then
		-- Try to create leaderstats if it doesn't exist (for local player)
		if player == LocalPlayer then
			leaderstats = Instance.new("Folder")
			leaderstats.Name = "leaderstats"
			leaderstats.Parent = player
		else
			return false, "Player has no leaderstats folder"
		end
	end
	
	local stat = leaderstats:FindFirstChild(statName)
	if not stat then
		-- Try to create the stat if it doesn't exist (for local player)
		if player == LocalPlayer then
			stat = Instance.new("IntValue")
			stat.Name = statName
			stat.Parent = leaderstats
		else
			return false, "Stat '" .. statName .. "' not found"
		end
	end
	
	-- Store original value if not already stored
	local playerKey = tostring(player.UserId)
	if not originalStats[playerKey] then
		originalStats[playerKey] = {}
	end
	if originalStats[playerKey][statName] == nil then
		originalStats[playerKey][statName] = stat.Value
	end
	
	-- Set the new value
	local success, err = pcall(function()
		stat.Value = newValue
	end)
	
	if success then
		-- Log to debug instead of print
		log(LOG_LEVEL.DEBUG, "[StatSpoof] Set %s.%s to %s", player.Name, statName, formatStatNumber(newValue))
		return true
	else
		return false, "Failed to set value: " .. tostring(err)
	end
end

-- Revert a player's spoofed stats to original values
local function revertLeaderstats(player)
	if not player then return false, "Player not found" end
	
	local playerKey = tostring(player.UserId)
	if not originalStats[playerKey] then
		return false, "No original stats stored for this player"
	end
	
	local leaderstats = player:FindFirstChild("leaderstats")
	if not leaderstats then
		return false, "Player has no leaderstats folder"
	end
	
	local revertedCount = 0
	for statName, originalValue in pairs(originalStats[playerKey]) do
		local stat = leaderstats:FindFirstChild(statName)
		if stat then
			pcall(function()
				stat.Value = originalValue
				revertedCount = revertedCount + 1
			end)
		end
	end
	
	-- Clear stored originals
	originalStats[playerKey] = nil
	
	return true, string.format("Reverted %d stats", revertedCount)
end

-- Monitor and maintain spoofed stats (in case game tries to reset them)
local spoofMonitors = {}

local function startStatMonitor(player, statName, spoofedValue)
	local playerKey = tostring(player.UserId) .. "_" .. statName
	
	-- Stop existing monitor if any
	if spoofMonitors[playerKey] then
		spoofMonitors[playerKey]:Disconnect()
		spoofMonitors[playerKey] = nil
	end
	
	local leaderstats = player:FindFirstChild("leaderstats")
	if not leaderstats then return end
	
	local stat = leaderstats:FindFirstChild(statName)
	if not stat then return end
	
	-- Monitor for changes and reapply spoofed value
	spoofMonitors[playerKey] = stat:GetPropertyChangedSignal("Value"):Connect(function()
		if stat.Value ~= spoofedValue then
			task.defer(function()
				pcall(function()
					stat.Value = spoofedValue
				end)
			end)
		end
	end)
end

local function stopStatMonitor(player, statName)
	local playerKey = tostring(player.UserId) .. "_" .. statName
	if spoofMonitors[playerKey] then
		spoofMonitors[playerKey]:Disconnect()
		spoofMonitors[playerKey] = nil
	end
end

local function stopAllStatMonitors(player)
	local prefix = tostring(player.UserId) .. "_"
	local toRemove = {}
	for key, connection in pairs(spoofMonitors) do
		if key:sub(1, #prefix) == prefix then
			connection:Disconnect()
			table.insert(toRemove, key)
		end
	end
	for _, key in ipairs(toRemove) do
		spoofMonitors[key] = nil
	end
end

local function log(level, message, ...)
	if level <= currentLogLevel then
		local prefix = ""
		if level == LOG_LEVEL.ERROR then
			prefix = "[ERROR] "
		elseif level == LOG_LEVEL.WARN then
			prefix = "[WARN] "
		elseif level == LOG_LEVEL.INFO then
			prefix = "[INFO] "
		elseif level == LOG_LEVEL.DEBUG then
			prefix = "[DEBUG] "
		end

		local formattedMessage = string.format(message, ...)
		print(prefix .. formattedMessage)
	end
end

-- Get HumanoidDescription with caching
local function getCachedHumanoidDescription(userId)
	local key = getCacheKey(userId, "humdesc")
	local cached = humanoidDescCache[key]
	if cached and isCacheValid(cached) then
		return cached.data
	end

	local success, result = pcall(function()
		return Players:GetHumanoidDescriptionFromUserId(userId)
	end)

	if success and result then
		setCachedHumanoidDescription(userId, result)
		return result
	end

	return nil
end

-- Executor function detection
local sethiddenprop = sethiddenproperty or set_hidden_property or sethiddenprop or nil
local gethiddenprop = gethiddenproperty or get_hidden_property or gethiddenprop or nil

-- Alternative: some executors use setrawproperty
if not sethiddenprop then
	sethiddenprop = setrawproperty or set_raw_property or nil
end

-- Check if we have executor capabilities
local hasExecutor = (sethiddenprop ~= nil) or (typeof(getgenv) == "function")

-- Function to set a property (handles read-only with executor)
local function setProp(object, property, value)
	if sethiddenprop then
		local success = pcall(function()
			sethiddenprop(object, property, value)
		end)
		if success then return true end
	end
	
	-- Try direct assignment
	local success = pcall(function()
		object[property] = value
	end)
	return success
end

-- Get CharacterAppearance URL from userId
local function getCharacterAppearanceUrl(userId)
	return "https://assetdelivery.roblox.com/v1/asset/?userID=" .. tostring(userId)
end

-- Store original player properties before modifying
local function storeOriginalPlayerData(player)
	if originalPlayerData[player.UserId] then 

		return 
	end
	
	local char = player.Character
	local humanoid = char and char:FindFirstChildWhichIsA("Humanoid")
	
	originalPlayerData[player.UserId] = {
		odUserId = player.UserId,
		name = player.Name,
		displayName = player.DisplayName,
		characterAppearanceId = player.CharacterAppearanceId,
		humanoidDisplayName = humanoid and humanoid.DisplayName or player.DisplayName
	}
	
	-- Original data stored
end

-- Get avatar appearance model from a userId with caching
local function getAvatarModel(userId)
	local cached = getCachedAvatarModel(userId)
	if cached then
		return cached, nil
	end

	local success, result = pcall(function()
		return Players:GetCharacterAppearanceAsync(userId)
	end)
	if not success then
		local errorMsg = "Failed to get avatar data for user ID " .. tostring(userId) .. ": " .. tostring(result)
		warn(errorMsg)
		return nil, errorMsg
	end
	if not result then
		local errorMsg = "GetCharacterAppearanceAsync returned nil for user ID " .. tostring(userId)
		warn(errorMsg)
		return nil, errorMsg
	end

	setCachedAvatarModel(userId, result)
	return result, nil
end

-- Validate UserID input
local function validateUserId(userIdStr)
	if not userIdStr or userIdStr == "" then
		return false, "UserID cannot be empty"
	end

	local userId = tonumber(userIdStr)
	if not userId then
		return false, "UserID must be a number"
	end

	-- Check if it's a valid Roblox UserID range (Roblox UserIDs start from 1)
	if userId < 1 or userId > 10^12 then
		return false, "Invalid UserID range"
	end

	-- Check if it's an integer
	if userId ~= math.floor(userId) then
		return false, "UserID must be a whole number"
	end

	return true, nil, userId
end

-- Get player info (username and display name) with caching
local function getPlayerInfo(userId)
	local cachedUsername, cachedDisplayName = getCachedUserInfo(userId)
	if cachedUsername and cachedDisplayName then
		print("[getPlayerInfo] Using cached data for", userId, ":", cachedUsername, "/", cachedDisplayName)
		return cachedUsername, cachedDisplayName
	end

	local username = nil
	local displayName = nil
	
	print("[getPlayerInfo] Fetching info for userId:", userId)

	-- Method 1: Get username from GetNameFromUserIdAsync
	local ok, res = pcall(function()
		return Players:GetNameFromUserIdAsync(userId)
	end)
	if ok and typeof(res) == "string" and res ~= "" then
		username = res
	end
	
	-- Method 2: Try to get from player in game (if they're in the server)
	ok, res = pcall(function()
		local player = Players:GetPlayerByUserId(userId)
		if player then
			return player.DisplayName, player.Name
		end
		return nil, nil
	end)
	if ok then
		if res and res ~= "" then
			displayName = res
		end
	end
	
	if not displayName or displayName == "" then
		ok, res = pcall(function()
			local UserService = game:GetService("UserService")
			local userInfos = UserService:GetUserInfosByUserIdsAsync({userId})
			if userInfos and userInfos[1] then
				return userInfos[1]
			end
			return nil
		end)
		if ok and res then
			if not username and res.Username then
				username = res.Username
			end
			if res.DisplayName and res.DisplayName ~= "" then
				displayName = res.DisplayName
			end
		end
	end
	
	-- Method 4: Try to get username from HumanoidDescription (backup)
	if not username or username == "" then
		ok, res = pcall(function()
			local desc = Players:GetHumanoidDescriptionFromUserId(userId)
			-- HumanoidDescription doesn't have username, but if we get here without error
			-- it means the userId is valid
			return true
		end)
		if ok and res then
			print("[getPlayerInfo] Method 4: UserId", userId, "is valid but couldn't get name")
		end
	end
	
	-- Method 5: HTTP request fallback (if all else fails)
	if not username or username == "" or not displayName or displayName == "" then
		ok, res = pcall(function()
			-- Try request function if available (executor feature)
			local requestFunc = request or http_request or (syn and syn.request) or (http and http.request)
			if requestFunc then
				local response = requestFunc({
					Url = "https://users.roblox.com/v1/users/" .. tostring(userId),
					Method = "GET"
				})
				if response and response.Body then
					local data = HttpService:JSONDecode(response.Body)
					return data
				end
			end
			return nil
		end)
		if ok and res then
			if not username and res.name then
				username = res.name
				print("[getPlayerInfo] Method 5 (HTTP) got username:", username)
			end
			if (not displayName or displayName == "") and res.displayName then
				displayName = res.displayName
				print("[getPlayerInfo] Method 5 (HTTP) got displayName:", displayName)
			end
		else
			print("[getPlayerInfo] Method 5 failed:", res)
		end
	end
	
	-- Final fallback: if we have username but no displayName, use username as displayName
	if username and (not displayName or displayName == "") then
		displayName = username
		print("[getPlayerInfo] Using username as displayName fallback")
	end
	
	-- EMERGENCY: If still no username, try to construct from userId
	if not username or username == "" then
		warn("[getPlayerInfo] WARNING: Could not fetch username for userId:", userId)
		warn("[getPlayerInfo] All API methods failed - name changes will not work")
		-- Return nil to signal failure
		return nil, nil
	end
	
	-- Cache the result
	setCachedUserInfo(userId, username, displayName)
	
	return username, displayName
end

-- Get avatar items from HumanoidDescription for examine window interception
local function getAvatarItemsFromUserId(userId)
	local humanoidDesc = getCachedHumanoidDescription(userId)

	if not humanoidDesc then
		return nil
	end
	
	local items = {}
	
	-- Get accessory IDs
	local accessoryTypes = {
		"BackAccessory", "FaceAccessory", "FrontAccessory", "HairAccessory",
		"HatAccessory", "NeckAccessory", "ShouldersAccessory", "WaistAccessory"
	}
	
	for _, accessoryType in ipairs(accessoryTypes) do
		local accessoryIds = humanoidDesc[accessoryType]
		if accessoryIds and accessoryIds ~= "" then
			for id in string.gmatch(tostring(accessoryIds), "[^,]+") do
				local numId = tonumber(id:match("%d+"))
				if numId then
					table.insert(items, {type = "Accessory", id = numId, category = accessoryType})
				end
			end
		end
	end
	
	-- Get body parts
	local bodyParts = {
		"Head", "Torso", "RightArm", "LeftArm", "RightLeg", "LeftLeg"
	}
	for _, part in ipairs(bodyParts) do
		local partId = humanoidDesc[part]
		if partId and partId ~= 0 then
			table.insert(items, {type = "BodyPart", id = partId, category = part})
		end
	end
	
	-- Get clothing
	if humanoidDesc.Shirt and humanoidDesc.Shirt ~= 0 then
		table.insert(items, {type = "Shirt", id = humanoidDesc.Shirt})
	end
	if humanoidDesc.Pants and humanoidDesc.Pants ~= 0 then
		table.insert(items, {type = "Pants", id = humanoidDesc.Pants})
	end
	if humanoidDesc.GraphicTShirt and humanoidDesc.GraphicTShirt ~= 0 then
		table.insert(items, {type = "TShirt", id = humanoidDesc.GraphicTShirt})
	end
	
	-- Get face
	if humanoidDesc.Face and humanoidDesc.Face ~= 0 then
		table.insert(items, {type = "Face", id = humanoidDesc.Face})
	end
	
	return items
end

-- Helper functions
local function getHumanoid(char)
	return char:FindFirstChildWhichIsA("Humanoid")
end

local function clearCosmetics(char)
	for _, item in ipairs(char:GetChildren()) do
		if item:IsA("Accessory") then
			item:Destroy()
		elseif item:IsA("Shirt") or item:IsA("Pants") or item:IsA("ShirtGraphic") then
			item:Destroy()
		elseif item:IsA("BodyColors") then
			item:Destroy()
		end
	end
end

local function copyClothingAndColors(fromModel, toChar)
	for _, item in ipairs(fromModel:GetChildren()) do
		if item:IsA("Shirt") or item:IsA("Pants") or item:IsA("ShirtGraphic") or item:IsA("BodyColors") then
			item:Clone().Parent = toChar
		end
	end
end

-- Copy dynamic head features (FaceControls, Wrap deformers, etc.)
local function copyDynamicHeadFeatures(srcHead, dstHead)
	if not srcHead or not dstHead then return end
	
	-- Remove existing dynamic head features from destination
	for _, child in ipairs(dstHead:GetChildren()) do
		if child:IsA("FaceControls") or child:IsA("WrapTarget") or child:IsA("WrapLayer") then
			child:Destroy()
		end
	end
	
	-- Copy FaceControls (for facial animations on dynamic heads)
	local srcFaceControls = srcHead:FindFirstChildOfClass("FaceControls")
	if srcFaceControls then
		local newFaceControls = srcFaceControls:Clone()
		newFaceControls.Parent = dstHead
	end
	
	-- Copy WrapTarget and WrapLayer (for skin/clothing deformation)
	for _, child in ipairs(srcHead:GetChildren()) do
		if child:IsA("WrapTarget") or child:IsA("WrapLayer") then
			local cloned = child:Clone()
			cloned.Parent = dstHead
		end
	end
	
	-- Copy any SurfaceAppearance (for PBR materials on heads)
	local srcSurfaceAppearance = srcHead:FindFirstChildOfClass("SurfaceAppearance")
	if srcSurfaceAppearance then
		local existingSA = dstHead:FindFirstChildOfClass("SurfaceAppearance")
		if existingSA then
			existingSA:Destroy()
		end
		local newSA = srcSurfaceAppearance:Clone()
		newSA.Parent = dstHead
	end
end

local function copyBodyMeshes(fromModel, toChar, humanoid)
	if not humanoid then return end
	
	local rigType = humanoid.RigType
	
	-- Copy R15 body parts using ReplaceBodyPartR15
	if rigType == Enum.HumanoidRigType.R15 then
		for _, src in ipairs(fromModel:GetChildren()) do
			if src:IsA("BasePart") and src.Name ~= "HumanoidRootPart" then
				local bodyPartName = src.Name
				local bodyPartEnum = nil
				
				-- Map common body part names to R15 enum
				local bodyPartMap = {
					["Head"] = "Head",
					["UpperTorso"] = "UpperTorso",
					["LowerTorso"] = "LowerTorso",
					["LeftUpperArm"] = "LeftUpperArm",
					["LeftLowerArm"] = "LeftLowerArm",
					["LeftHand"] = "LeftHand",
					["RightUpperArm"] = "RightUpperArm",
					["RightLowerArm"] = "RightLowerArm",
					["RightHand"] = "RightHand",
					["LeftUpperLeg"] = "LeftUpperLeg",
					["LeftLowerLeg"] = "LeftLowerLeg",
					["LeftFoot"] = "LeftFoot",
					["RightUpperLeg"] = "RightUpperLeg",
					["RightLowerLeg"] = "RightLowerLeg",
					["RightFoot"] = "RightFoot",
				}
				
				bodyPartEnum = bodyPartMap[bodyPartName]
				
				if bodyPartEnum then
					-- Special handling for Head (dynamic heads)
					if bodyPartName == "Head" and src:IsA("MeshPart") then
						local dstHead = toChar:FindFirstChild("Head")
						if dstHead and dstHead:IsA("MeshPart") then
							-- Copy mesh and texture
							pcall(function()
								dstHead.MeshId = src.MeshId
								dstHead.TextureID = src.TextureID
							end)
							-- Copy dynamic head features
							copyDynamicHeadFeatures(src, dstHead)
							-- Copy size for proper scaling
							pcall(function()
								dstHead.Size = src.Size
							end)
						end
					else
						local success = pcall(function()
							humanoid:ReplaceBodyPartR15(Enum.BodyPartR15[bodyPartEnum], src)
						end)
						if not success then
							-- Fallback: try to copy mesh properties if it's a MeshPart
							local dst = toChar:FindFirstChild(bodyPartName)
							if dst and dst:IsA("MeshPart") and src:IsA("MeshPart") then
								pcall(function()
									dst.MeshId = src.MeshId
									dst.TextureID = src.TextureID
									-- Also copy size for body proportions
									dst.Size = src.Size
								end)
							end
						end
					end
				end
			end
		end
	-- Copy R6 body parts using ReplaceBodyPartR6
	elseif rigType == Enum.HumanoidRigType.R6 then
		for _, src in ipairs(fromModel:GetChildren()) do
			if src:IsA("BasePart") and src.Name ~= "HumanoidRootPart" then
				local bodyPartName = src.Name
				local bodyPartEnum = nil
				
				-- Map R6 body part names to R6 enum
				local bodyPartMap = {
					["Head"] = "Head",
					["Torso"] = "Torso",
					["Left Arm"] = "LeftArm",
					["Right Arm"] = "RightArm",
					["Left Leg"] = "LeftLeg",
					["Right Leg"] = "RightLeg",
				}
				
				bodyPartEnum = bodyPartMap[bodyPartName]
				
				if bodyPartEnum then
					local success = pcall(function()
						humanoid:ReplaceBodyPartR6(Enum.BodyPartR6[bodyPartEnum], src)
					end)
					if not success then
						-- Fallback: directly replace the part
						local dst = toChar:FindFirstChild(bodyPartName)
						if dst and dst:IsA("BasePart") then
							-- Clone the source part
							local cloned = src:Clone()
							cloned.Name = bodyPartName
							cloned.CFrame = dst.CFrame
							cloned.Anchored = dst.Anchored
							cloned.CanCollide = dst.CanCollide
							
							-- Copy over any special properties
							if dst:IsA("MeshPart") and cloned:IsA("MeshPart") then
								cloned.MeshId = src.MeshId
								cloned.TextureID = src.TextureID
							end
							
							-- Transfer welds and connections
							for _, weld in ipairs(dst:GetChildren()) do
								if weld:IsA("Weld") or weld:IsA("WeldConstraint") then
									local newWeld = weld:Clone()
									if newWeld.Part0 == dst then
										newWeld.Part0 = cloned
									end
									if newWeld.Part1 == dst then
										newWeld.Part1 = cloned
									end
									newWeld.Parent = cloned
								end
							end
							
							-- Replace the part
							dst:Destroy()
							cloned.Parent = toChar
						end
					end
				end
			end
		end
	end
	
	-- Also copy mesh properties for any MeshParts (R6 or R15) as fallback
	for _, src in ipairs(fromModel:GetChildren()) do
		if src:IsA("MeshPart") then
			local dst = toChar:FindFirstChild(src.Name)
			if dst and dst:IsA("MeshPart") then
				-- Only update if not already replaced
				if dst.MeshId ~= src.MeshId or dst.TextureID ~= src.TextureID then
					pcall(function()
						dst.MeshId = src.MeshId
						dst.TextureID = src.TextureID
					end)
				end
				-- Copy SurfaceAppearance if exists
				local srcSA = src:FindFirstChildOfClass("SurfaceAppearance")
				if srcSA then
					local existingSA = dst:FindFirstChildOfClass("SurfaceAppearance")
					if existingSA then existingSA:Destroy() end
					srcSA:Clone().Parent = dst
				end
				-- Copy WrapTarget/WrapLayer for layered clothing
				for _, child in ipairs(src:GetChildren()) do
					if child:IsA("WrapTarget") or child:IsA("WrapLayer") then
						local existing = dst:FindFirstChild(child.Name)
						if existing then existing:Destroy() end
						child:Clone().Parent = dst
					end
				end
			end
		end
	end
end

-- Copy face separately using multiple methods
local function copyFace(fromUserId, toChar, avatarModel)
	local dstHead = toChar:FindFirstChild("Head")
	if not dstHead or not dstHead:IsA("BasePart") then
		return
	end
	
	-- Remove existing face first
	local existingFace = dstHead:FindFirstChild("face")
	if not existingFace then
		for _, d in ipairs(dstHead:GetChildren()) do
			if d:IsA("Decal") and d.Name:lower() == "face" then
				existingFace = d
				break
			end
		end
	end
	if existingFace then
		existingFace:Destroy()
	end
	
	-- Method 1: Try to find face in the avatar model first (most direct)
	local srcFace = nil
	if avatarModel then
		local srcHead = avatarModel:FindFirstChild("Head")
		if srcHead and srcHead:IsA("BasePart") then
			-- Search for face decal in source head
			srcFace = srcHead:FindFirstChild("face")
			if not srcFace then
				-- Try case-insensitive search
				for _, d in ipairs(srcHead:GetChildren()) do
					if d:IsA("Decal") and d.Name:lower() == "face" then
						srcFace = d
						break
					end
				end
			end
			
			-- Also check in all descendants
			if not srcFace then
				for _, d in ipairs(srcHead:GetDescendants()) do
					if d:IsA("Decal") and d.Name:lower() == "face" then
						srcFace = d
						break
					end
				end
			end
		end
		
		-- Also check the model itself (sometimes face is at model level)
		if not srcFace then
			for _, d in ipairs(avatarModel:GetDescendants()) do
				if d:IsA("Decal") and d.Name:lower() == "face" then
					srcFace = d
					break
				end
			end
		end
	end
	
	-- If found in model, clone it
	if srcFace and srcFace:IsA("Decal") and srcFace.Texture then
		local newFace = srcFace:Clone()
		newFace.Name = "face"
		newFace.Parent = dstHead
		-- Ensure it's visible
		task.wait(0.05)
		if not newFace.Parent then
			newFace.Parent = dstHead
		end
		return
	end
	
	-- Method 2: Create a model from HumanoidDescription to get the face
	local success, tempModel = pcall(function()
		local humanoidDesc = Players:GetHumanoidDescriptionFromUserId(fromUserId)
		return Players:CreateHumanoidModelFromDescription(humanoidDesc, Enum.HumanoidRigType.R15)
	end)
	
	if success and tempModel then
		local tempHead = tempModel:FindFirstChild("Head")
		if tempHead then
			local tempFace = tempHead:FindFirstChild("face")
			if not tempFace then
				for _, d in ipairs(tempHead:GetDescendants()) do
					if d:IsA("Decal") and d.Name:lower() == "face" then
						tempFace = d
						break
					end
				end
			end
			
			if tempFace and tempFace:IsA("Decal") and tempFace.Texture then
				local newFace = tempFace:Clone()
				newFace.Name = "face"
				newFace.Parent = dstHead
				tempModel:Destroy()
				return
			end
		end
		if tempModel then
			tempModel:Destroy()
		end
	end
	
	-- Method 3: Try to get face ID from HumanoidDescription directly
	success, humanoidDesc = pcall(function()
		return Players:GetHumanoidDescriptionFromUserId(fromUserId)
	end)
	
	if success and humanoidDesc then
		local faceId = humanoidDesc.Face
		if faceId and faceId ~= 0 then
			-- Create new face with the texture
			local newFace = Instance.new("Decal")
			newFace.Name = "face"
			-- Use proper texture format
			if type(faceId) == "number" then
				newFace.Texture = "rbxassetid://" .. tostring(faceId)
			else
				newFace.Texture = tostring(faceId)
			end
			newFace.Face = Enum.NormalId.Front
			newFace.Parent = dstHead
			-- Ensure it's visible
			task.wait(0.05)
			if not newFace.Parent then
				newFace.Parent = dstHead
			end
			return
		end
	end
	
	-- If all else fails, keep the default face (don't remove it)
	warn("Could not copy face for user ID:", fromUserId)
end

-- Copy body scales and proportions from HumanoidDescription
local function copyBodyScales(fromUserId, toHumanoid)
	if not toHumanoid then return end

	local humanoidDesc = getCachedHumanoidDescription(fromUserId)
	if not humanoidDesc then return end

	-- Only apply conservative body scaling to avoid visual glitches
	-- Skip HeadScale as it often causes issues with face positioning
	local safeScaleProperties = {
		"BodyTypeScale", "DepthScale", "HeightScale",
		"ProportionScale", "WidthScale"
	}

	local scaleApplied = false
	for _, prop in ipairs(safeScaleProperties) do
		pcall(function()
			local value = humanoidDesc[prop]
			if value and value ~= 0 and value >= 0.5 and value <= 2.0 then
				-- Only apply reasonable scale values to prevent extreme distortion
				local scaleValue = toHumanoid:FindFirstChild(prop)
				if scaleValue and scaleValue:IsA("NumberValue") then
					scaleValue.Value = value
					scaleApplied = true
				end
			end
		end)
	end

	-- Only try ApplyDescription for non-head properties if we applied some scales
	if scaleApplied then
		pcall(function()
			-- Create a copy of the description with head scale removed to avoid issues
			local safeDesc = humanoidDesc:Clone()
			safeDesc.HeadScale = 1  -- Reset to default
			toHumanoid:ApplyDescription(safeDesc)
		end)
	end
end

-- Copy animations from source to target (R15 focused)
local function copyAnimations(fromUserId, toHumanoid)
	if not toHumanoid then return end
	
	-- Only apply animations for R15 rigs
	if toHumanoid.RigType ~= Enum.HumanoidRigType.R15 then return end
	
	local success, humanoidDesc = pcall(function()
		return Players:GetHumanoidDescriptionFromUserId(fromUserId)
	end)
	
	if not success or not humanoidDesc then return end
	
	local char = toHumanoid.Parent
	if not char then return end
	
	-- Get or create Animator
	local animator = toHumanoid:FindFirstChildOfClass("Animator")
	if not animator then
		animator = Instance.new("Animator")
		animator.Parent = toHumanoid
	end
	
	-- Get animation IDs from HumanoidDescription
	local animationIds = {
		Idle = humanoidDesc.IdleAnimation,
		Walk = humanoidDesc.WalkAnimation,
		Run = humanoidDesc.RunAnimation,
		Jump = humanoidDesc.JumpAnimation,
		Fall = humanoidDesc.FallAnimation,
		Climb = humanoidDesc.ClimbAnimation,
		Swim = humanoidDesc.SwimAnimation,
	}
	
	-- Find or create the Animate script's animation folder
	local animateScript = char:FindFirstChild("Animate")
	
	if animateScript then
		-- Update existing animation references
		for animName, animId in pairs(animationIds) do
			if animId and animId ~= 0 then
				local animFolder = animateScript:FindFirstChild(animName:lower())
				if animFolder then
					for _, child in ipairs(animFolder:GetChildren()) do
						if child:IsA("Animation") then
							pcall(function()
								child.AnimationId = "rbxassetid://" .. tostring(animId)
							end)
						end
					end
				end
			end
		end
	end
	
	-- Get emotes and apply them
	local success2, emotes = pcall(function()
		return humanoidDesc:GetEmotes()
	end)
	
	if success2 and emotes then
		-- Preload emote animations
		local emoteIds = {}
		for emoteName, emoteTable in pairs(emotes) do
			if type(emoteTable) == "table" then
				for _, emoteId in ipairs(emoteTable) do
					if emoteId and emoteId ~= 0 then
						table.insert(emoteIds, "rbxassetid://" .. tostring(emoteId))
					end
				end
			end
		end
		
		if #emoteIds > 0 then
			pcall(function()
				game:GetService("ContentProvider"):PreloadAsync(emoteIds)
			end)
		end
	end
	
	-- Try to apply the humanoid description for animations (works best for local player)
	-- This sets up the animation controller properly
	local char = toHumanoid.Parent
	if char and char == LocalPlayer.Character then
		-- For local player, we can apply description which sets up animations
		pcall(function()
			-- Get current description and update animations only
			local currentDesc = toHumanoid:GetAppliedDescription() or Instance.new("HumanoidDescription")
			
			-- Copy animation IDs
			currentDesc.IdleAnimation = humanoidDesc.IdleAnimation
			currentDesc.WalkAnimation = humanoidDesc.WalkAnimation
			currentDesc.RunAnimation = humanoidDesc.RunAnimation
			currentDesc.JumpAnimation = humanoidDesc.JumpAnimation
			currentDesc.FallAnimation = humanoidDesc.FallAnimation
			currentDesc.ClimbAnimation = humanoidDesc.ClimbAnimation
			currentDesc.SwimAnimation = humanoidDesc.SwimAnimation
			
			-- Apply emotes
			pcall(function()
				local emotes = humanoidDesc:GetEmotes()
				for emoteName, emoteIds in pairs(emotes) do
					currentDesc:SetEmotes({[emoteName] = emoteIds})
				end
			end)
			
			-- Apply equipped emotes
			pcall(function()
				local equippedEmotes = humanoidDesc:GetEquippedEmotes()
				if equippedEmotes and #equippedEmotes > 0 then
					currentDesc:SetEquippedEmotes(equippedEmotes)
				end
			end)
			
			toHumanoid:ApplyDescription(currentDesc)
		end)
	end
end

-- Get body scale values from a humanoid
local function getBodyScales(humanoid)
	local scales = {
		BodyHeightScale = 1,
		BodyWidthScale = 1,
		BodyDepthScale = 1,
		HeadScale = 1,
		BodyProportionScale = 0,
		BodyTypeScale = 0,
	}
	
	if not humanoid then return scales end
	
	-- Read scale values from Humanoid's NumberValue children
	for scaleName, defaultValue in pairs(scales) do
		local scaleValue = humanoid:FindFirstChild(scaleName)
		if scaleValue and scaleValue:IsA("NumberValue") then
			scales[scaleName] = scaleValue.Value
		end
	end
	
	return scales
end

-- Get body scales from HumanoidDescription
local function getBodyScalesFromDescription(humanoidDesc)
	local scales = {
		BodyHeightScale = 1,
		BodyWidthScale = 1,
		BodyDepthScale = 1,
		HeadScale = 1,
		BodyProportionScale = 0,
		BodyTypeScale = 0,
	}
	
	if not humanoidDesc then return scales end
	
	pcall(function()
		scales.BodyHeightScale = humanoidDesc.HeightScale or 1
		scales.BodyWidthScale = humanoidDesc.WidthScale or 1
		scales.BodyDepthScale = humanoidDesc.DepthScale or 1
		scales.HeadScale = humanoidDesc.HeadScale or 1
		scales.BodyProportionScale = humanoidDesc.ProportionScale or 0
		scales.BodyTypeScale = humanoidDesc.BodyTypeScale or 0
	end)
	
	return scales
end

-- Calculate scale adjustment for accessories based on body part
local function calculateAccessoryScale(attachmentName, targetScales, sourceScales)
	-- Default scale
	local scaleX, scaleY, scaleZ = 1, 1, 1
	
	-- Determine which body scales to use based on attachment name
	local attachLower = attachmentName:lower()
	
	-- Head attachments (hats, hair, face accessories)
	if attachLower:find("hat") or attachLower:find("hair") or attachLower:find("head") or attachLower:find("face") then
		local headRatio = targetScales.HeadScale / math.max(sourceScales.HeadScale, 0.01)
		scaleX = headRatio
		scaleY = headRatio
		scaleZ = headRatio
	-- Body/Torso attachments (backpacks, chest items)
	elseif attachLower:find("body") or attachLower:find("torso") or attachLower:find("chest") or attachLower:find("back") or attachLower:find("waist") or attachLower:find("neck") then
		local widthRatio = targetScales.BodyWidthScale / math.max(sourceScales.BodyWidthScale, 0.01)
		local heightRatio = targetScales.BodyHeightScale / math.max(sourceScales.BodyHeightScale, 0.01)
		local depthRatio = targetScales.BodyDepthScale / math.max(sourceScales.BodyDepthScale, 0.01)
		scaleX = widthRatio
		scaleY = heightRatio
		scaleZ = depthRatio
	-- Shoulder/Arm attachments
	elseif attachLower:find("shoulder") or attachLower:find("arm") or attachLower:find("hand") then
		local widthRatio = targetScales.BodyWidthScale / math.max(sourceScales.BodyWidthScale, 0.01)
		scaleX = widthRatio
		scaleY = widthRatio
		scaleZ = widthRatio
	-- Leg/Foot attachments
	elseif attachLower:find("leg") or attachLower:find("foot") then
		local heightRatio = targetScales.BodyHeightScale / math.max(sourceScales.BodyHeightScale, 0.01)
		scaleX = heightRatio
		scaleY = heightRatio
		scaleZ = heightRatio
	else
		-- Default: use average of all scales
		local avgTarget = (targetScales.BodyHeightScale + targetScales.BodyWidthScale + targetScales.HeadScale) / 3
		local avgSource = (sourceScales.BodyHeightScale + sourceScales.BodyWidthScale + sourceScales.HeadScale) / 3
		local ratio = avgTarget / math.max(avgSource, 0.01)
		scaleX = ratio
		scaleY = ratio
		scaleZ = ratio
	end
	
	-- Clamp scales to reasonable values to prevent extreme distortion
	scaleX = math.clamp(scaleX, 0.5, 2.0)
	scaleY = math.clamp(scaleY, 0.5, 2.0)
	scaleZ = math.clamp(scaleZ, 0.5, 2.0)
	
	return Vector3.new(scaleX, scaleY, scaleZ)
end

local function copyAccessories(fromModel, humanoid, targetChar, sourceUserId)
	-- Get target body scales
	local targetScales = getBodyScales(humanoid)
	
	-- Get source body scales from HumanoidDescription if we have sourceUserId
	local sourceScales = {
		BodyHeightScale = 1, BodyWidthScale = 1, BodyDepthScale = 1,
		HeadScale = 1, BodyProportionScale = 0, BodyTypeScale = 0
	}
	
	if sourceUserId then
		local success, sourceDesc = pcall(function()
			return Players:GetHumanoidDescriptionFromUserId(sourceUserId)
		end)
		if success and sourceDesc then
			sourceScales = getBodyScalesFromDescription(sourceDesc)
		end
	end
	
	for _, item in ipairs(fromModel:GetChildren()) do
		if item:IsA("Accessory") then
			local cloned = item:Clone()
			local handle = cloned:FindFirstChild("Handle")
			
			if not handle then
				warn("Accessory has no Handle:", cloned.Name)
				cloned:Destroy()
			else
				-- Find the attachment in the handle
				local handleAttachment = handle:FindFirstChildOfClass("Attachment")
				if not handleAttachment then
					warn("Accessory handle has no Attachment:", cloned.Name)
					cloned:Destroy()
				else
					-- Get the attachment name to find matching body part attachment
					local attachmentName = handleAttachment.Name
					
					-- Find the matching attachment on the character
					local bodyPart = nil
					local bodyAttachment = nil
					
					-- Search through all body parts for matching attachment
					for _, part in ipairs(targetChar:GetChildren()) do
						if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
							local attachment = part:FindFirstChild(attachmentName)
							if attachment and attachment:IsA("Attachment") then
								bodyPart = part
								bodyAttachment = attachment
								break
							end
						end
					end
					
					-- If no matching attachment found, try to determine from accessory name
					if not bodyPart then
						local accessoryName = cloned.Name:lower()
						if accessoryName:find("hat") or accessoryName:find("hair") or accessoryName:find("cap") or accessoryName:find("helmet") or accessoryName:find("head") then
							bodyPart = targetChar:FindFirstChild("Head")
						elseif accessoryName:find("torso") or accessoryName:find("chest") or accessoryName:find("backpack") or accessoryName:find("shirt") then
							bodyPart = targetChar:FindFirstChild("UpperTorso") or targetChar:FindFirstChild("Torso")
						elseif accessoryName:find("leftarm") or (accessoryName:find("left") and accessoryName:find("arm")) then
							bodyPart = targetChar:FindFirstChild("LeftUpperArm") or targetChar:FindFirstChild("Left Arm")
						elseif accessoryName:find("rightarm") or (accessoryName:find("right") and accessoryName:find("arm")) then
							bodyPart = targetChar:FindFirstChild("RightUpperArm") or targetChar:FindFirstChild("Right Arm")
						elseif accessoryName:find("leftleg") or (accessoryName:find("left") and accessoryName:find("leg")) then
							bodyPart = targetChar:FindFirstChild("LeftUpperLeg") or targetChar:FindFirstChild("Left Leg")
						elseif accessoryName:find("rightleg") or (accessoryName:find("right") and accessoryName:find("leg")) then
							bodyPart = targetChar:FindFirstChild("RightUpperLeg") or targetChar:FindFirstChild("Right Leg")
						else
							-- Default to head
							bodyPart = targetChar:FindFirstChild("Head")
						end
						
						-- Try to find attachment on the body part
						if bodyPart then
							bodyAttachment = bodyPart:FindFirstChild(attachmentName)
							if not bodyAttachment then
								bodyAttachment = bodyPart:FindFirstChild("AccessoryAttachment") or bodyPart:FindFirstChildOfClass("Attachment")
							end
						end
					end
					
					if not bodyPart then
						warn("Could not find body part for accessory:", cloned.Name)
						cloned:Destroy()
					else
						if not bodyAttachment then
							-- Create a default attachment at origin
							bodyAttachment = Instance.new("Attachment")
							bodyAttachment.Name = attachmentName
							bodyAttachment.Parent = bodyPart
						end
						
						-- Calculate scale adjustment for this accessory
						local scaleAdjust = calculateAccessoryScale(attachmentName, targetScales, sourceScales)
						
						-- Apply scale to handle if scales differ significantly
						if math.abs(scaleAdjust.X - 1) > 0.05 or math.abs(scaleAdjust.Y - 1) > 0.05 or math.abs(scaleAdjust.Z - 1) > 0.05 then
							-- Scale the handle mesh/part
							pcall(function()
								if handle:IsA("MeshPart") then
									-- For MeshParts, we can adjust the size
									local originalSize = handle.Size
									handle.Size = Vector3.new(
										originalSize.X * scaleAdjust.X,
										originalSize.Y * scaleAdjust.Y,
										originalSize.Z * scaleAdjust.Z
									)
								elseif handle:IsA("Part") then
									local mesh = handle:FindFirstChildOfClass("SpecialMesh")
									if mesh then
										-- Scale the mesh
										mesh.Scale = mesh.Scale * scaleAdjust
									else
										-- Scale the part
										handle.Size = handle.Size * scaleAdjust
									end
								end
								
								-- Adjust attachment position in handle
								local handleAtt = handle:FindFirstChildOfClass("Attachment")
								if handleAtt then
									handleAtt.Position = handleAtt.Position * scaleAdjust
								end
							end)
						end
						
						-- Set parent first
						cloned.Parent = targetChar
						
						-- Remove any existing Motor6D
						for _, existingMotor in ipairs(handle:GetChildren()) do
							if existingMotor:IsA("Motor6D") then
								existingMotor:Destroy()
							end
						end
						
						-- Create Motor6D to attach
						local motor = Instance.new("Motor6D")
						motor.Name = "AccessoryWeld"
						motor.Part0 = bodyPart
						motor.Part1 = handle
						
						-- Calculate proper C0 and C1 from attachments
						if bodyAttachment and handleAttachment then
							-- Get base CFrames
							local c0 = bodyAttachment.CFrame
							local c1 = handleAttachment.CFrame
							
							-- Apply scale adjustment to C1 offset
							if math.abs(scaleAdjust.Y - 1) > 0.05 then
								-- Adjust vertical offset based on height scale
								local adjustedC1Pos = c1.Position * scaleAdjust
								c1 = CFrame.new(adjustedC1Pos) * (c1 - c1.Position)
							end
							
							motor.C0 = c0
							motor.C1 = c1
						else
							motor.C0 = CFrame.new()
							motor.C1 = CFrame.new()
						end
						
						motor.Parent = handle
					end
				end
			end
		end
	end
end

-- Change player name to match source
local function changePlayerName(toPlayer, newUsername, newDisplayName, sourceUserId)
	-- Get original name from the stored data (set BEFORE any name changes happen)
	local storedOriginal = originalPlayerData[toPlayer.UserId]
	
	-- Store original name for reference (use stored data if available)
	if not toPlayer:GetAttribute("OriginalName") then
		local origName = storedOriginal and storedOriginal.name or toPlayer.Name
		local origDisplayName = storedOriginal and storedOriginal.displayName or toPlayer.DisplayName
		toPlayer:SetAttribute("OriginalName", origName)
		toPlayer:SetAttribute("OriginalDisplayName", origDisplayName)
	end
	
	-- Store the fake names (source player's names)
	toPlayer:SetAttribute("FakeUsername", newUsername)
	if newDisplayName and newDisplayName ~= "" then
		toPlayer:SetAttribute("FakeDisplayName", newDisplayName)
	else
		toPlayer:SetAttribute("FakeDisplayName", newUsername) -- Use username as display name if no display name
	end
	
	local fakeUsername = newUsername
	-- ALWAYS use display name if provided, otherwise use username
	-- Note: Display name CAN be the same as username, that's valid
	local fakeDisplayName = (newDisplayName and newDisplayName ~= "") and newDisplayName or newUsername
	local char = toPlayer.Character
	
	-- Use the ORIGINAL values (before any changes were made)
	local originalName = toPlayer:GetAttribute("OriginalName")
	local originalDisplayName = toPlayer:GetAttribute("OriginalDisplayName")
	
	-- CRITICAL: If original names are nil/empty, the replacement won't work
	-- Fall back to current player names if needed
	if not originalName or originalName == "" then
		originalName = storedOriginal and storedOriginal.name or "UNKNOWN"
		warn("WARNING: OriginalName was nil, using fallback:", originalName)
	end
	if not originalDisplayName or originalDisplayName == "" then
		originalDisplayName = storedOriginal and storedOriginal.displayName or originalName
		warn("WARNING: OriginalDisplayName was nil, using fallback:", originalDisplayName)
	end
	
	-- Debug output removed for performance
	
	-- Update existing name tag instead of creating fake one
	local function updateExistingNameTag(char)
		if not char then return end
		
		local humanoidRootPart = char:FindFirstChild("HumanoidRootPart")
		local head = char:FindFirstChild("Head")
		
		-- Find and update existing name tags
		for _, part in ipairs({humanoidRootPart, head}) do
			if part then
				for _, child in ipairs(part:GetChildren()) do
					if child:IsA("BillboardGui") then
						local textLabel = child:FindFirstChild("TextLabel")
						if textLabel then
							local currentText = textLabel.Text
							-- If it shows the original name or display name, update it
							if currentText == originalName or currentText == originalDisplayName or
							   currentText == toPlayer.Name or currentText == toPlayer.DisplayName then
								textLabel.Text = fakeDisplayName
							end
						end
					end
				end
			end
		end
		
		-- Also check character model itself
		for _, child in ipairs(char:GetChildren()) do
			if child:IsA("BillboardGui") then
				local textLabel = child:FindFirstChild("TextLabel")
				if textLabel then
					local currentText = textLabel.Text
					if currentText == originalName or currentText == originalDisplayName or
					   currentText == toPlayer.Name or currentText == toPlayer.DisplayName then
						textLabel.Text = fakeDisplayName
					end
				end
			end
		end
	end
	
	updateExistingNameTag(char)

	-- Chat Monitoring System
	local function setupChatMonitoring()
		-- Handle TextChatService (New Chat)
		local TextChatService = game:GetService("TextChatService")
		local chatEvents = TextChatService:FindFirstChild("ChatWindowConfiguration")
		
		-- Monitor new chat messages
		local function processChatMessage(messageObj)
			if not messageObj then return end
			-- We mainly check the UI as we can't easily modify the message object itself for others
		end
		
		-- Scan Chat UI for messages
		local function scanChatUI()
			local playerGui = LocalPlayer:WaitForChild("PlayerGui")
			if not playerGui then return end
			
			-- Legacy Chat
			local chatGui = playerGui:FindFirstChild("Chat")
			if chatGui then
				local scroller = chatGui:FindFirstChild("Scroller", true) or chatGui:FindFirstChild("ChatChannelParentFrame", true)
				if scroller then
					for _, msg in ipairs(scroller:GetDescendants()) do
						if msg:IsA("TextLabel") or msg:IsA("TextButton") then
							local text = msg.Text
							-- Check patterns like "[Name]: Message" or "Name: Message"
							if text:find(originalName, 1, true) or text:find(originalDisplayName, 1, true) then
								-- Do replacement
								local newText = text:gsub(originalName, fakeUsername):gsub(originalDisplayName, fakeDisplayName)
								if newText ~= text then
									msg.Text = newText
								end
							end
						end
					end
				end
			end
			
			-- TextChatService UI (ExpChat)
			local expChat = playerGui:FindFirstChild("ExperienceChat")
			if expChat then
				for _, descendant in ipairs(expChat:GetDescendants()) do
					if descendant:IsA("TextLabel") then
						local text = descendant.Text
						-- Look for the name
						if text == originalName or text == originalDisplayName then
							descendant.Text = fakeDisplayName
						elseif text:find(originalName .. ":") or text:find(originalDisplayName .. ":") then
							local newText = text:gsub(originalName, fakeDisplayName):gsub(originalDisplayName, fakeDisplayName)
							descendant.Text = newText
						end
					end
				end
			end
		end
		
		-- Hook into UI updates
		local chatTimer = RunService.Heartbeat:Connect(function()
			if tick() % 1 < 0.1 then -- Run once per second roughly
				scanChatUI()
			end
		end)
		table.insert(activeConnections[toPlayer.UserId], chatTimer)
	end
	
	setupChatMonitoring()
	
	-- Function to check if a UI element belongs to the TARGET player (not other players)
	local function isElementForTargetPlayer(element)
		if not element or not element.Parent then return false end
		
		-- Check the element and its ancestors for player identification
		local current = element
		local foundTargetReference = false
		local foundOtherPlayerReference = false
		
		-- Search up to 10 levels up the hierarchy
		for _ = 1, 10 do
			if not current or not current.Parent then break end
			
			-- Check for UserId attribute on frames
			local frameUserId = current:GetAttribute("UserId") or current:GetAttribute("PlayerUserId") or current:GetAttribute("userid")
			if frameUserId then
				if frameUserId == toPlayer.UserId then
					foundTargetReference = true
					break
				else
					foundOtherPlayerReference = true
					break
				end
			end
			
			-- Check for player name in the frame/element name
			local elementName = current.Name:lower()
			if elementName:find(tostring(toPlayer.UserId), 1, true) then
				foundTargetReference = true
				break
			end
			
			-- Check sibling text elements for the target player's name
			if current.Parent then
				for _, sibling in ipairs(current.Parent:GetChildren()) do
					if sibling ~= current and (sibling:IsA("TextLabel") or sibling:IsA("TextButton")) then
						local siblingText = sibling.Text or ""
						-- Check if sibling shows target player's original OR fake name
						if siblingText == originalName or siblingText == originalDisplayName or
						   siblingText == fakeUsername or siblingText == fakeDisplayName or
						   siblingText == "@" .. originalName or siblingText == "@" .. fakeUsername then
							foundTargetReference = true
						end
					end
				end
			end
			
			current = current.Parent
		end
		
		-- If we found a reference to another player, don't update
		if foundOtherPlayerReference then
			return false
		end
		
		-- If we found a reference to the target player, update
		if foundTargetReference then
			return true
		end
		
		-- If we couldn't determine, be conservative - only update if text exactly matches
		-- the ORIGINAL name (not a common name that might belong to other players)
		return false
	end
	
	-- Function to update text elements in UI (ONLY for target player)
	local function updateTextElement(element)
		if not element or not element.Parent then return false end
		if not (element:IsA("TextLabel") or element:IsA("TextButton") or element:IsA("TextBox")) then return false end
		
		local text = element.Text
		if not text or text == "" then return false end
		
		local textTrimmed = text:gsub("^%s+", ""):gsub("%s+$", "")
		
		-- Define the @ prefixed versions
		local atOriginal = "@" .. (originalName or "")
		local atOriginalDisplay = "@" .. (originalDisplayName or "")
		local atFake = "@" .. fakeUsername
		
		-- CRITICAL: Only process if this element is for the TARGET player
		-- Check if text matches original name EXACTLY first
		local isExactOriginalMatch = (textTrimmed == originalName) or 
		                              (textTrimmed == originalDisplayName) or
		                              (textTrimmed == atOriginal) or
		                              (textTrimmed == atOriginalDisplay)
		
		-- If it's not an exact match to the original, verify this element belongs to target player
		if not isExactOriginalMatch then
			-- For @ fields, we need to be extra careful
			if textTrimmed:sub(1, 1) == "@" then
				-- Only replace @ fields if we can confirm it's for the target player
				if not isElementForTargetPlayer(element) then
					return false
				end
			else
				-- Skip if it doesn't match original names
				return false
			end
		end
		
		-- Now we're sure this is for the target player, apply replacements
		
		-- RULE 1: If text exactly matches @original, replace with @fake
		if textTrimmed == atOriginal or textTrimmed == atOriginalDisplay then
			element.Text = atFake
			return true
		end
		
		-- RULE 2: If text exactly matches original display name
		if textTrimmed == originalDisplayName then
			local verified = toPlayer:GetAttribute("ShowVerified")
			local premium = toPlayer:GetAttribute("ShowPremium")
			local suffix = ""
			
			if verified or premium then
				if element.RichText then
					if verified then suffix = suffix .. " <font color='#00a2ff'>[âœ“]</font>" end
					if premium then suffix = suffix .. " <font color='#e6e6e6'>[P]</font>" end
				else
					if verified then suffix = suffix .. " âœ“" end
					if premium then suffix = suffix .. " â’¿" end
				end
			end
			
			element.Text = fakeDisplayName .. suffix
			return true
		end
		
		-- RULE 3: If text exactly matches original username
		if textTrimmed == originalName then
			element.Text = fakeUsername
			return true
		end
		
		-- RULE 4: For @ fields that passed the target player check
		if textTrimmed:sub(1, 1) == "@" and textTrimmed ~= atFake then
			-- Double check this is actually for our player by verifying context
			if isElementForTargetPlayer(element) then
				local verified = toPlayer:GetAttribute("ShowVerified")
				local premium = toPlayer:GetAttribute("ShowPremium")
				local suffix = ""
				
				-- Prepare suffix
				if verified or premium then
					if element.RichText then
						if verified then suffix = suffix .. " <font color='#00a2ff'>[âœ“]</font>" end
						if premium then suffix = suffix .. " <font color='#e6e6e6'>[P]</font>" end
					else
						-- Fallback for non-rich text
						if verified then suffix = suffix .. " âœ“" end
						if premium then suffix = suffix .. " â’¿" end
					end
				end
				
				element.Text = atFake .. suffix
				return true
			end
		end

		return false
	end
	
	-- Update player list UI
	local function updatePlayerList()
		local playerGui = LocalPlayer:WaitForChild("PlayerGui")
		
		-- Search all descendants in PlayerGui
		for _, descendant in ipairs(playerGui:GetDescendants()) do
			updateTextElement(descendant)
		end
		
		-- Also check CoreGui for player list
		local coreGui = game:GetService("CoreGui")
		for _, descendant in ipairs(coreGui:GetDescendants()) do
			updateTextElement(descendant)
		end
		
		-- Try to find player entry by UserId
		local function findPlayerEntryByUserId(gui, userId)
			for _, frame in ipairs(gui:GetDescendants()) do
				if frame:IsA("Frame") or frame:IsA("TextButton") then
					local frameUserId = frame:GetAttribute("UserId") or frame:GetAttribute("PlayerUserId")
					if frameUserId == userId then
						for _, child in ipairs(frame:GetDescendants()) do
							updateTextElement(child)
						end
					end
					for _, child in ipairs(frame:GetChildren()) do
						local childUserId = child:GetAttribute("UserId") or child:GetAttribute("PlayerUserId")
						if childUserId == userId then
							for _, textChild in ipairs(frame:GetDescendants()) do
								updateTextElement(textChild)
							end
						end
					end
				end
			end
		end
		
		findPlayerEntryByUserId(playerGui, toPlayer.UserId)
		findPlayerEntryByUserId(game:GetService("CoreGui"), toPlayer.UserId)
	end
	
	-- Run immediately
	updatePlayerList()
	
	-- Store monitoring state
	toPlayer:SetAttribute("MonitoringActive", true)
	activeConnections[toPlayer.UserId] = {}
	
	print("=== Starting UI Monitoring ===")
	print("Will monitor for original name:", originalName)
	print("Will replace with fake name:", fakeUsername, "/ display:", fakeDisplayName)
	
	-- Aggressive monitoring system
	local function monitorUI()
		local playerGui = LocalPlayer:WaitForChild("PlayerGui")
		local coreGui = game:GetService("CoreGui")
		
		local function restoreFakeNames(element)
			if not element or not element.Parent then return end
			updateTextElement(element)
		end
		
		local function setupTextListener(element)
			if element:IsA("TextLabel") or element:IsA("TextButton") or element:IsA("TextBox") then
				restoreFakeNames(element)
				local textConnection = element:GetPropertyChangedSignal("Text"):Connect(function()
					restoreFakeNames(element)
				end)
				table.insert(activeConnections[toPlayer.UserId], textConnection)
			end
		end
		
		local descendantConnection = playerGui.DescendantAdded:Connect(function(descendant)
			setupTextListener(descendant)
		end)
		table.insert(activeConnections[toPlayer.UserId], descendantConnection)
		
		for _, descendant in ipairs(playerGui:GetDescendants()) do
			setupTextListener(descendant)
		end
		
		-- Monitor CoreGui
		local function setupCoreTextListener(element)
			if element:IsA("TextLabel") or element:IsA("TextButton") or element:IsA("TextBox") then
				restoreFakeNames(element)
				local textConnection = element:GetPropertyChangedSignal("Text"):Connect(function()
					restoreFakeNames(element)
				end)
				table.insert(activeConnections[toPlayer.UserId], textConnection)
			end
		end
		
		local coreDescendantConnection = coreGui.DescendantAdded:Connect(function(descendant)
			setupCoreTextListener(descendant)
		end)
		table.insert(activeConnections[toPlayer.UserId], coreDescendantConnection)
		
		for _, descendant in ipairs(coreGui:GetDescendants()) do
			setupCoreTextListener(descendant)
		end
		
		-- Optimized heartbeat monitoring - check every 0.5 seconds instead of every frame
		local lastHeartbeatUpdate = 0
		local heartbeatConnection = RunService.Heartbeat:Connect(function()
			if toPlayer:GetAttribute("MonitoringActive") ~= true then
				heartbeatConnection:Disconnect()
				return
			end

			local currentTime = tick()
			if currentTime - lastHeartbeatUpdate >= 0.5 then  -- Reduced frequency
				lastHeartbeatUpdate = currentTime

				-- More selective scanning - only check elements that are likely to change
				for _, descendant in ipairs(playerGui:GetDescendants()) do
					if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
						local elementName = descendant.Name:lower()
						-- Only check elements that might contain player names
						if elementName:find("player") or elementName:find("name") or elementName:find("display") or elementName == "" then
							restoreFakeNames(descendant)
						end
					end
				end

				for _, descendant in ipairs(coreGui:GetDescendants()) do
					if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
						local elementName = descendant.Name:lower()
						if elementName:find("player") or elementName:find("name") or elementName:find("display") or elementName == "" then
							restoreFakeNames(descendant)
						end
					end
				end
			end
		end)
		table.insert(activeConnections[toPlayer.UserId], heartbeatConnection)
		
		-- Handle Tab and ESC keys
		local inputBeganConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
			if gameProcessed then return end
			
			if input.KeyCode == Enum.KeyCode.Tab or input.KeyCode == Enum.KeyCode.Escape then
				task.spawn(function()
					updatePlayerList()
					task.wait(0.1)
					updatePlayerList()
					task.wait(0.2)
					updatePlayerList()
					task.wait(0.3)
					updatePlayerList()
				end)
			end
		end)
		table.insert(activeConnections[toPlayer.UserId], inputBeganConnection)
	end
	
	task.spawn(monitorUI)
	
	-- No name tag creation - removed as requested
end

-- Set up avatar interception for profiles and player list (defined early so it can be called)
local function setupAvatarInterception(toPlayer: Player, sourceUserId: number)
	if not toPlayer or not sourceUserId then return end
	
	local playerGui = LocalPlayer:WaitForChild("PlayerGui")
	local coreGui = game:GetService("CoreGui")
	local targetUserId = toPlayer.UserId
	
	-- Build thumbnail URL patterns to detect and replace
	local function getAvatarThumbnailUrl(userId: number, thumbType: string, size: number): string
		size = size or 150
		return "rbxthumb://type=" .. thumbType .. "&id=" .. tostring(userId) .. "&w=" .. tostring(size) .. "&h=" .. tostring(size)
	end
	
	-- Common thumbnail types used in player list/menus
	local thumbnailTypes = {"AvatarHeadShot", "AvatarThumbnail", "Avatar", "AvatarBust"}
	local thumbnailSizes = {48, 50, 60, 100, 150, 180, 352, 420}
	
	-- Function to replace avatar image URLs in ImageLabels/ImageButtons
	local function replaceAvatarImageUrl(imageElement)
		if not imageElement or not imageElement.Parent then return false end
		if not (imageElement:IsA("ImageLabel") or imageElement:IsA("ImageButton")) then return false end
		
		local currentImage = imageElement.Image
		if not currentImage or currentImage == "" then return false end
		
		-- Check if this image contains the TARGET player's UserId (the one we're replacing)
		local targetIdStr = tostring(targetUserId)
		if currentImage:find(targetIdStr, 1, true) then
			-- Replace target UserId with source UserId in the URL
			local newImage = currentImage:gsub(targetIdStr, tostring(sourceUserId))
			if newImage ~= currentImage then
				imageElement.Image = newImage
				return true
			end
		end
		
		return false
	end
	
	-- Function to update avatar in viewports (profiles, player list, etc.)
	local function updateViewportAvatar(viewport: ViewportFrame)
		if not viewport or not viewport.Parent then return false end
		
		-- Get humanoid description first - ALWAYS use sourceUserId
		local success, humanoidDesc = pcall(function()
			return Players:GetHumanoidDescriptionFromUserId(sourceUserId)
		end)
		
		if not success or not humanoidDesc then 
			return false
		end
		
		-- Check if this viewport has a character model to determine rig type
		local charModel = viewport:FindFirstChildOfClass("Model")
		local rigType = Enum.HumanoidRigType.R15
		
		if charModel then
			local humanoid = charModel:FindFirstChildOfClass("Humanoid")
			if humanoid then
				rigType = humanoid.RigType or Enum.HumanoidRigType.R15
			end
		end
		
		-- Create a new model from the SOURCE user's description
		local success2, newModel = pcall(function()
			return Players:CreateHumanoidModelFromDescription(humanoidDesc, rigType)
		end)
		
		if not success2 or not newModel then 
			return false
		end
		
		-- Clear ALL old characters/models FIRST - be thorough
		local childrenToRemove = {}
		for _, child in ipairs(viewport:GetChildren()) do
			if child:IsA("Model") or child:IsA("BasePart") then
				table.insert(childrenToRemove, child)
			end
		end
		for _, child in ipairs(childrenToRemove) do
			pcall(function()
				child:Destroy()
			end)
		end
		
		-- Wait a tiny bit to ensure cleanup
		task.wait(0.01)
		
		-- Mark as our model and add to viewport
		newModel:SetAttribute("SourceAvatarCopy", true)
		newModel.Parent = viewport
		
		-- Position camera - try multiple angles for player list
		if viewport.CurrentCamera then
			local humanoidRootPart = newModel:FindFirstChild("HumanoidRootPart")
			if humanoidRootPart then
				-- Try different camera positions (player list uses different angles)
				local cameraPositions = {
					CFrame.new(humanoidRootPart.Position + Vector3.new(0, 2, 5), humanoidRootPart.Position),
					CFrame.new(humanoidRootPart.Position + Vector3.new(0, 1, 3), humanoidRootPart.Position),
					CFrame.new(humanoidRootPart.Position + Vector3.new(2, 1, 3), humanoidRootPart.Position),
				}
				viewport.CurrentCamera.CFrame = cameraPositions[1]
				-- Also try setting it again after a moment
				task.spawn(function()
					task.wait(0.05)
					if viewport.Parent and newModel.Parent == viewport then
						viewport.CurrentCamera.CFrame = cameraPositions[1]
					end
				end)
			end
		end
		
		return true
	end
	
	-- Cache source avatar items for examine window
	if not sourceAvatarItems[sourceUserId] then
		sourceAvatarItems[sourceUserId] = getAvatarItemsFromUserId(sourceUserId)
	end
	
-- Roblox Avatar Impersonation Script
-- Complete avatar copying with inspect menu interception

-- ============================================================
	-- AVATAR INSPECT MENU INTERCEPTION (Using Official GuiService API)
	-- ============================================================
	-- Uses GuiService:InspectPlayerFromHumanoidDescription() to show
	-- the SOURCE player's avatar/items when inspecting the TARGET player.
	-- This shows source's avatar but with target's name displayed.

-- Get player info (username and display name)

-- Service declarations
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")
local LocalPlayer = Players.LocalPlayer
local coreGui = game:GetService("CoreGui")
local playerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Store active monitoring connections per player
local activeConnections = {}

-- Store source avatar item data for examine window interception
local sourceAvatarItems = {}

	local lastInspectRedirectTime = 0
	local cachedSourceHumanoidDesc = nil
	
	-- Get the source user's HumanoidDescription (cached)
	local function getSourceHumanoidDescription()
		if cachedSourceHumanoidDesc then
			print("Using cached source description for user", sourceUserId)
			return cachedSourceHumanoidDesc
		end

		print("Fetching fresh source description for user", sourceUserId)
		local success, desc = pcall(function()
			return Players:GetHumanoidDescriptionFromUserId(sourceUserId)
		end)

		if success and desc then
			cachedSourceHumanoidDesc = desc
			print("Successfully cached source description")
			return desc
		else
			warn("Failed to get source description:", desc)
		end

		return nil
	end
	
	-- Open inspect menu showing SOURCE's avatar but with TARGET's name
	local function openInspectWithSourceAvatar()
		-- Stronger debounce to prevent reopening
		local now = tick()
		if now - lastInspectRedirectTime < 1.0 then return false end  -- Increased to 1 second
		lastInspectRedirectTime = now

		-- Only open if we're not already in an inspect menu
		if isInspectMenuForTargetPlayer() then
			print("Inspect menu already open for target player")
			return false  -- Don't open if already open
		end

		print("Opening inspect menu for target player:", toPlayer.Name, "(showing source:", sourceUserId, ")")

		-- Get the source user's HumanoidDescription
		local sourceDesc = getSourceHumanoidDescription()
		if not sourceDesc then
			warn("Could not get source HumanoidDescription for user", sourceUserId)
			return false
		end

		-- Get the source player's username to show (for complete impersonation)
		local sourceUsername, sourceDisplayName = getPlayerInfo(sourceUserId)
		local displayName = sourceUsername or tostring(sourceUserId)
		print("Using source username for inspect menu:", displayName)

		-- Use official API: Shows SOURCE's avatar items but with TARGET's name
		local success = pcall(function()
			print("Calling GuiService.InspectPlayerFromHumanoidDescription with source desc and target name")
			GuiService:InspectPlayerFromHumanoidDescription(sourceDesc, displayName)
		end)

		if success then
			print("Successfully opened inspect menu with source avatar for:", displayName)
		else
			warn("Failed to open inspect menu")
		end

		return success
	end
	
	-- Check if inspect menu is currently open for the target player
	-- More specific detection to avoid confusing Tab/ESC menu with inspect menu
	local function isInspectMenuForTargetPlayer(): boolean
		local targetName = toPlayer.Name:lower()
		local targetDisplayName = toPlayer.DisplayName:lower()
		local fakeUsername = (toPlayer:GetAttribute("FakeUsername") or ""):lower()
		local fakeDisplayName = (toPlayer:GetAttribute("FakeDisplayName") or ""):lower()

		-- Search CoreGui for SPECIFIC inspect menu elements (not player list)
		for _, descendant in ipairs(coreGui:GetDescendants()) do
			if descendant:IsA("TextLabel") or descendant:IsA("TextButton") then
				local text = (descendant.Text or ""):lower()
				if text ~= "" and (text:find(targetName, 1, true) or text:find(targetDisplayName, 1, true) or
					text:find(fakeUsername, 1, true) or text:find(fakeDisplayName, 1, true)) then
					-- Check if it's in a SPECIFIC inspect-related container (NOT player list/tab menu)
					local parent = descendant.Parent
					local isInspectMenu = false
					local isPlayerList = false
					
					while parent and parent ~= coreGui do
						local parentName = parent.Name:lower()
						-- Check for player list/tab menu (should NOT trigger inspect)
						if parentName:find("playerlist") or parentName:find("leaderboard") or 
						   parentName:find("topbar") or parentName:find("people") or
						   parentName:find("corescripts") then
							isPlayerList = true
							break
						end
						-- Check for actual inspect menu
						if parentName:find("inspectmenu") or parentName:find("avatarcard") or 
						   parentName:find("inspectplayerfrom") then
							isInspectMenu = true
							break
						end
						parent = parent.Parent
					end
					
					if isInspectMenu and not isPlayerList then
						return true
					end
				end
			end
		end

		return false
	end
	
	-- Setup inspect menu interception using GuiService hook (clean, non-interfering method)
	local function setupInspectMenuRedirect()
		-- Hook GuiService.InspectPlayerFromHumanoidDescription to intercept inspect calls
		-- This is the cleanest way - it only triggers when inspect is actually called
		local originalInspectFunc = GuiService.InspectPlayerFromHumanoidDescription
		if originalInspectFunc then
			GuiService.InspectPlayerFromHumanoidDescription = function(humanoidDesc, displayName)
				-- Check if this inspect call is for our target player
				local targetUsername = toPlayer:GetAttribute("FakeUsername") or toPlayer.Name
				local targetDisplayName = toPlayer:GetAttribute("FakeDisplayName") or toPlayer.DisplayName
				if displayName == targetUsername or displayName == targetDisplayName or displayName == toPlayer.Name then
					-- This is our target player being inspected - redirect to source avatar
					print("Intercepted inspect for target player:", displayName, "- showing source avatar")
					local sourceDesc = getSourceHumanoidDescription()
					if sourceDesc then
						-- Get source player's username for display
						local sourceUsername, _ = getPlayerInfo(sourceUserId)
						local newDisplayName = sourceUsername or tostring(sourceUserId)

						local success = pcall(function()
							return originalInspectFunc(sourceDesc, newDisplayName)
						end)
						if success then
							print("Successfully redirected to source avatar with name:", newDisplayName)
							return
						end
					else
						warn("Could not get source description for interception")
					end
				end
				-- Not our target, call original
				return originalInspectFunc(humanoidDesc, displayName)
			end

			-- Store the hook so we can restore it later
			if not activeConnections[toPlayer.UserId] then
				activeConnections[toPlayer.UserId] = {}
			end
			table.insert(activeConnections[toPlayer.UserId], {
				Disconnect = function()
					GuiService.InspectPlayerFromHumanoidDescription = originalInspectFunc
				end
			})
		end
	end
	
	-- Function to handle when user tries to inspect the target player
	local function handleExamineWindow()
		openInspectWithSourceAvatar()
	end
	
	-- Setup the inspect menu redirect system
	local function setupExamineWindowMonitor()
		setupInspectMenuRedirect()

		-- Pre-cache the source HumanoidDescription
		task.spawn(function()
			getSourceHumanoidDescription()
		end)

		-- DISABLED: Click monitoring was causing Tab menu to close
		-- The GuiService hook handles inspect interception cleanly without interfering
		-- No additional monitoring needed - the GuiService hook is sufficient
	end
	
	-- Function to replace avatar image URLs in ImageLabels/ImageButtons
	local function replaceAvatarImageUrl(imageElement)
		if not imageElement or not imageElement.Parent then return false end
		if not (imageElement:IsA("ImageLabel") or imageElement:IsA("ImageButton")) then return false end

		local currentImage = imageElement.Image
		if not currentImage or currentImage == "" then return false end

		-- Check if this image contains the TARGET player's UserId (the one we're replacing)
		local targetIdStr = tostring(targetUserId)
		if currentImage:find(targetIdStr, 1, true) then
			-- Replace target UserId with source UserId in the URL
			local newImage = currentImage:gsub(targetIdStr, tostring(sourceUserId))
			if newImage ~= currentImage then
				imageElement.Image = newImage
				print("Replaced avatar thumbnail:", targetIdStr, "->", tostring(sourceUserId))
				return true
			end
		end

		return false
	end

	-- Scan and replace all avatar images in a parent
	local function scanAndReplaceAvatarImages(parent)
		for _, descendant in ipairs(parent:GetDescendants()) do
			if descendant:IsA("ImageLabel") or descendant:IsA("ImageButton") then
				pcall(function()
					replaceAvatarImageUrl(descendant)
				end)
			end
		end
	end
	
	-- Monitor Image property changes and replace them back
	local function setupImageMonitor(imageElement)
		if not imageElement or not imageElement.Parent then return end
		if not (imageElement:IsA("ImageLabel") or imageElement:IsA("ImageButton")) then return end
		
		local connection = imageElement:GetPropertyChangedSignal("Image"):Connect(function()
			task.defer(function()
				replaceAvatarImageUrl(imageElement)
			end)
		end)
		
		if not activeConnections[toPlayer.UserId] then
			activeConnections[toPlayer.UserId] = {}
		end
		table.insert(activeConnections[toPlayer.UserId], connection)
		
		-- Also replace immediately
		replaceAvatarImageUrl(imageElement)
	end
	
	
	-- Monitor for new viewports AND images (profiles, player list, etc.)
	local function monitorViewports()
		-- FIRST: Replace all avatar IMAGE URLs in CoreGui (player list, ESC menu)
		-- This is the PRIMARY method for fixing player list avatars
		pcall(function()
			scanAndReplaceAvatarImages(coreGui)
		end)
		pcall(function()
			scanAndReplaceAvatarImages(playerGui)
		end)
		
		-- Update ALL viewports aggressively - update EVERY viewport we find
		-- This catches name tags, player list, inspection window, etc.
		local viewportCount = 0
		
		-- Update PlayerGui viewports
		for _, descendant in ipairs(playerGui:GetDescendants()) do
			if descendant:IsA("ViewportFrame") then
				viewportCount = viewportCount + 1
				pcall(function()
					updateViewportAvatar(descendant)
				end)
			end
		end
		
		-- CoreGui is where player list and ESC menu are
		for _, descendant in ipairs(coreGui:GetDescendants()) do
			if descendant:IsA("ViewportFrame") then
				viewportCount = viewportCount + 1
				pcall(function()
					-- Clear any existing models first
					for _, child in ipairs(descendant:GetChildren()) do
						if child:IsA("Model") then
							child:Destroy()
						end
					end
					-- Then update with source avatar
					updateViewportAvatar(descendant)
				end)
			end
		end
		
		-- Also update viewports multiple times to catch ones that get reset
		if viewportCount > 0 then
			task.spawn(function()
				task.wait(0.05)
				for _, descendant in ipairs(coreGui:GetDescendants()) do
					if descendant:IsA("ViewportFrame") then
						pcall(function()
							updateViewportAvatar(descendant)
						end)
					end
				end
				task.wait(0.1)
				for _, descendant in ipairs(coreGui:GetDescendants()) do
					if descendant:IsA("ViewportFrame") then
						pcall(function()
							updateViewportAvatar(descendant)
						end)
					end
				end
			end)
		end
	end
	
	-- Function to intercept "Inspect Avatar" / "View" button clicks - LESS AGGRESSIVE
	local function interceptInspectAvatar()
		-- Track buttons we've already hooked to prevent duplicates
		local hookedButtons = {}
		local lastButtonIntercept = 0

		local function checkForInspectButton(descendant)
			if hookedButtons[descendant] then return end

			if descendant:IsA("TextButton") or descendant:IsA("ImageButton") then
				local buttonText = ""
				if descendant:IsA("TextButton") then
					buttonText = (descendant.Text or ""):lower()
				end
				local buttonName = descendant.Name:lower()

				-- More conservative button detection
				local isInspectButton = false
				if buttonText ~= "" then
					isInspectButton = (buttonText:find("inspect") or buttonText:find("examine") or
					                   buttonText:find("view avatar") or buttonText == "view")
				end
				if buttonName:find("inspect") or buttonName:find("examine") or
				   buttonName:find("viewavatar") or buttonName:find("view") or
				   buttonName:find("avatar") then
					isInspectButton = true
				end

				-- Only hook buttons that are clearly in inspect-related containers
				local parent = descendant.Parent
				local inInspectContext = false
				while parent and parent ~= game do
					local parentName = parent.Name:lower()
					if parentName:find("inspect") or parentName:find("avatar") or parentName:find("menu") then
						inInspectContext = true
						break
					end
					parent = parent.Parent
				end

				if isInspectButton and inInspectContext then
					hookedButtons[descendant] = true

					local clickConnection = descendant.MouseButton1Click:Connect(function()
						local now = tick()
						if now - lastButtonIntercept < 0.5 then return end -- Stronger debounce
						lastButtonIntercept = now

						-- Check if the button is related to the target player
						local isForTargetPlayer = false
						local searchParent = descendant.Parent

						-- Search siblings and ancestors for target player name
						if searchParent then
							for _, sibling in ipairs(searchParent:GetDescendants()) do
								if sibling:IsA("TextLabel") or sibling:IsA("TextButton") then
									local text = (sibling.Text or ""):lower()
									local targetName = toPlayer.Name:lower()
									local targetDisplayName = toPlayer.DisplayName:lower()
									local fakeUsername = (toPlayer:GetAttribute("FakeUsername") or ""):lower()
									local fakeDisplayName = (toPlayer:GetAttribute("FakeDisplayName") or ""):lower()

									if text:find(targetName, 1, true) or text:find(targetDisplayName, 1, true) or
									   text:find(fakeUsername, 1, true) or text:find(fakeDisplayName, 1, true) then
										isForTargetPlayer = true
										break
									end
								end
							end
						end

						if isForTargetPlayer then
							-- Directly open inspect for SOURCE user using official API
							task.spawn(function()
								task.wait(0.1) -- Small delay to let default action start
								openInspectWithSourceAvatar()
							end)
						end
					end)

					if not activeConnections[toPlayer.UserId] then
						activeConnections[toPlayer.UserId] = {}
					end
					table.insert(activeConnections[toPlayer.UserId], clickConnection)
				end
			end
		end

		for _, descendant in ipairs(playerGui:GetDescendants()) do
			checkForInspectButton(descendant)
		end
		for _, descendant in ipairs(coreGui:GetDescendants()) do
			checkForInspectButton(descendant)
		end

		local function setupButtonListener(parent)
			local connection = parent.DescendantAdded:Connect(function(descendant)
				checkForInspectButton(descendant)
			end)
			if not activeConnections[toPlayer.UserId] then
				activeConnections[toPlayer.UserId] = {}
			end
			table.insert(activeConnections[toPlayer.UserId], connection)
		end
		setupButtonListener(playerGui)
		setupButtonListener(coreGui)
	end
	
	-- Monitor for new elements being added (viewports and images) - SELECTIVE
	local function setupDescendantListener(parent)
		local connection = parent.DescendantAdded:Connect(function(descendant)
			if descendant:IsA("ViewportFrame") then
				-- Only update viewports that are likely to be player avatars
				-- Avoid interfering with ESC menu or other system viewports
				task.spawn(function()
					task.wait(0.1)  -- Let viewport settle

					-- Check if this viewport is in a player-related context
					local inPlayerContext = false
					local current = descendant
					for _ = 1, 5 do  -- Check up to 5 levels up
						if current and current.Parent then
							current = current.Parent
							local name = current.Name:lower()
							if name:find("player") or name:find("avatar") or name:find("inspect") or
							   name:find("menu") or name:find("list") then
								inPlayerContext = true
								break
							end
						end
					end

					-- Only update if it's in a player context and not too small (avoid UI elements)
					if inPlayerContext and descendant.AbsoluteSize.X > 50 then
						pcall(function()
							for _, child in ipairs(descendant:GetChildren()) do
								if child:IsA("Model") then
									child:Destroy()
								end
							end
						end)
						pcall(function()
							updateViewportAvatar(descendant)
						end)
					end
				end)
			elseif descendant:IsA("ImageLabel") or descendant:IsA("ImageButton") then
				-- Set up monitoring for this image and replace immediately
				task.spawn(function()
					task.wait(0.02)  -- Small delay for image URL to be set
					setupImageMonitor(descendant)
				end)
			end
		end)
		if not activeConnections[toPlayer.UserId] then
			activeConnections[toPlayer.UserId] = {}
		end
		table.insert(activeConnections[toPlayer.UserId], connection)
	end
	
	setupDescendantListener(playerGui)
	setupDescendantListener(coreGui)
	interceptInspectAvatar()
	setupExamineWindowMonitor()
	
	-- Set up monitoring for ALL existing images in CoreGui
	for _, descendant in ipairs(coreGui:GetDescendants()) do
		if descendant:IsA("ImageLabel") or descendant:IsA("ImageButton") then
			setupImageMonitor(descendant)
		end
	end
	
	-- Optimized periodic monitoring - less aggressive, more efficient
	local lastUpdateTime = 0
	local updateInterval = 3.0  -- Increased from 0.25 to 3 seconds

	task.spawn(function()
		while toPlayer:GetAttribute("AvatarCopied") == true do
			local currentTime = tick()
			if currentTime - lastUpdateTime >= updateInterval then
				lastUpdateTime = currentTime

				-- Only scan for new images that might have appeared, not everything
				pcall(function()
					local newImages = {}
					for _, descendant in ipairs(coreGui:GetDescendants()) do
						if descendant:IsA("ImageLabel") or descendant:IsA("ImageButton") then
							if not descendant:GetAttribute("AvatarMonitored") then
								descendant:SetAttribute("AvatarMonitored", true)
								setupImageMonitor(descendant)
								table.insert(newImages, descendant)
							end
						end
					end

					-- Only update new images instead of scanning everything
					for _, img in ipairs(newImages) do
						replaceAvatarImageUrl(img)
					end
				end)

				-- Less aggressive viewport checking - only check viewports that might be player-related
				local viewportCount = 0
				for _, descendant in ipairs(coreGui:GetDescendants()) do
					if descendant:IsA("ViewportFrame") and viewportCount < 5 then  -- Limit to 5 viewports per check
						local parent = descendant.Parent
						local isPlayerRelated = false
						if parent then
							local parentName = parent.Name:lower()
							isPlayerRelated = parentName:find("player") or parentName:find("avatar") or parentName:find("menu")
						end

						if isPlayerRelated then
							viewportCount = viewportCount + 1
							pcall(function()
								updateViewportAvatar(descendant)
							end)
						end
					end
				end
			end

			task.wait(1.0)  -- Check every second instead of every 0.25 seconds
		end
	end)
	
	-- Initial scan after a short delay
	task.spawn(function()
		task.wait(0.5)
		monitorViewports()
	end)
	
	-- Handle TAB key press to update player list (ESC monitoring removed entirely)
	local inputConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
		if gameProcessed then return end
		if input.KeyCode == Enum.KeyCode.Tab then
			task.spawn(function()
				-- Wait a moment for the menu to actually open
				task.wait(0.1)

				-- Update avatars and thumbnails for TAB menu
				scanAndReplaceAvatarImages(coreGui)
				scanAndReplaceAvatarImages(playerGui)

				-- Update viewports and their thumbnails
				for _, descendant in ipairs(coreGui:GetDescendants()) do
					if descendant:IsA("ViewportFrame") then
						pcall(function()
							for _, child in ipairs(descendant:GetChildren()) do
								if child:IsA("Model") then
									child:Destroy()
								end
							end
							updateViewportAvatar(descendant)
						end)
					end
				end

				-- Multiple passes with delays to catch late-loading elements
				for i = 1, 3 do
					task.wait(0.05 * i)
					scanAndReplaceAvatarImages(coreGui)
					monitorViewports()
				end
			end)
		end
	end)
	
	-- Handle mouse clicks (opening menus, etc.) - VERY CONSERVATIVE
	local lastMouseCheck = 0
	local mouseConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			local now = tick()
			if now - lastMouseCheck < 0.5 then return end -- Debounce mouse checks
			lastMouseCheck = now

			task.spawn(function()
				task.wait(0.3)  -- Longer delay to let menus settle

				-- Only update images if we're not actively in an inspect menu interaction
				-- This prevents reopening menus that the user just closed
				local inspectOpen = false
				for _, descendant in ipairs(coreGui:GetDescendants()) do
					if descendant:IsA("ViewportFrame") then
						local parent = descendant.Parent
						while parent and parent ~= coreGui do
							local parentName = parent.Name:lower()
							if parentName:find("inspect") or parentName:find("avatar") or parentName:find("menu") then
								inspectOpen = true
								break
							end
							parent = parent.Parent
						end
						if inspectOpen then break end
					end
				end

				-- Only update thumbnails if no inspect menu is currently open
				if not inspectOpen then
					scanAndReplaceAvatarImages(coreGui)
					scanAndReplaceAvatarImages(playerGui)
				end
			end)
		end
	end)
	
	if not activeConnections[toPlayer.UserId] then
		activeConnections[toPlayer.UserId] = {}
	end
	table.insert(activeConnections[toPlayer.UserId], inputConnection)
	table.insert(activeConnections[toPlayer.UserId], mouseConnection)
end


-- Internal function to copy avatar without setting up connections (for respawn)
-- Check if a character is still the player's current character (prevents race conditions)
local function isCharacterValidForPlayer(char, player)
	if not char or not player then return false end
	return player.Character == char
end

local function applyAvatarToCharacter(fromUserId, char, humanoid, skipRollback)
	if not char then
		return false, "Character is nil"
	end
	if not humanoid then
		return false, "Humanoid is nil"
	end

	-- Get the player who owns this character for validity checking
	local player = Players:GetPlayerFromCharacter(char)
	if not player then
		return false, "Character has no associated player"
	end

	-- Note: Removed automatic rollback as it was causing issues on respawn
	-- The rollback would restore to the default white avatar instead of keeping partial progress
	local operationSuccessful = false
	
	local avatarModel, errorMsg = getAvatarModel(fromUserId)
	if not avatarModel then 
		return false, "Failed to get avatar model: " .. tostring(errorMsg or "Unknown error")
	end
	
	-- For better body part copying, also create a model from HumanoidDescription
	local tempModel = nil
	local humanoidDesc = getCachedHumanoidDescription(fromUserId)
	if humanoidDesc then
		local modelSuccess, createdModel = pcall(function()
			return Players:CreateHumanoidModelFromDescription(humanoidDesc, humanoid.RigType)
		end)
		if modelSuccess and createdModel then
			tempModel = createdModel
		end
	end
	
	-- Wrap each step in pcall to catch errors, with validity checks
	local success1, err1 = pcall(function()
		if not isCharacterValidForPlayer(char, player) then return false end
		clearCosmetics(char)
		return true
	end)
	if not success1 or err1 == false then
		if err1 == false then
			return false, "Character became invalid during copying (respawn occurred)"
		end
		warn("Error clearing cosmetics:", err1)
	end

	local success2, err2 = pcall(function()
		if not isCharacterValidForPlayer(char, player) then return false end
		copyClothingAndColors(avatarModel, char)
		return true
	end)
	if not success2 or err2 == false then
		if err2 == false then
			return false, "Character became invalid during copying (respawn occurred)"
		end
		warn("Error copying clothing:", err2)
	end

	local success3, err3 = pcall(function()
		if not isCharacterValidForPlayer(char, player) then return false end
		if not safeModeEnabled then
			copyBodyMeshes(tempModel or avatarModel, char, humanoid)
		else
			log(LOG_LEVEL.INFO, "Safe mode: Skipping body mesh replacement")
		end
		return true
	end)
	if not success3 or err3 == false then
		if err3 == false then
			return false, "Character became invalid during copying (respawn occurred)"
		end
		warn("Error copying body meshes:", err3)
	end
	
	local success4, err4 = pcall(function()
		if not isCharacterValidForPlayer(char, player) then return false end
		copyFace(fromUserId, char, avatarModel)
		return true
	end)
	if not success4 or err4 == false then
		if err4 == false then
			return false, "Character became invalid during copying (respawn occurred)"
		end
		warn("Error copying face:", err4)
	end

	local success5, err5 = pcall(function()
		if not isCharacterValidForPlayer(char, player) then return false end
		copyAccessories(avatarModel, humanoid, char, fromUserId)
		return true
	end)
	if not success5 or err5 == false then
		if err5 == false then
			return false, "Character became invalid during copying (respawn occurred)"
		end
		warn("Error copying accessories:", err5)
	end

	local success6, err6 = pcall(function()
		if not isCharacterValidForPlayer(char, player) then return false end
		if not safeModeEnabled then
			copyAnimations(fromUserId, humanoid)
		else
			log(LOG_LEVEL.INFO, "Safe mode: Skipping animation copying")
		end
		return true
	end)
	if not success6 or err6 == false then
		if err6 == false then
			return false, "Character became invalid during copying (respawn occurred)"
		end
		warn("Error copying animations:", err6)
	end

	-- Copy body scales and proportions
	local success7, err7 = pcall(function()
		if not safeModeEnabled then
			copyBodyScales(fromUserId, humanoid)
		else
			log(LOG_LEVEL.INFO, "Safe mode: Skipping body scaling")
		end
	end)
	if not success7 then
		warn("Error copying body scales:", err7)
	end
	
	-- Apply skin tone to all body parts
	local success8, err8 = pcall(function()
		local bodyColors = char:FindFirstChildOfClass("BodyColors")
		if bodyColors then
			-- Apply body colors to all parts for consistency
			for _, part in ipairs(char:GetChildren()) do
				if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
					-- Don't override parts that have textures
					if part:IsA("MeshPart") and (not part.TextureID or part.TextureID == "") then
						-- Apply appropriate color based on body part
						if part.Name == "Head" then
							part.Color = bodyColors.HeadColor3
						elseif part.Name:find("Arm") or part.Name:find("Hand") then
							if part.Name:find("Left") then
								part.Color = bodyColors.LeftArmColor3
							else
								part.Color = bodyColors.RightArmColor3
							end
						elseif part.Name:find("Leg") or part.Name:find("Foot") then
							if part.Name:find("Left") then
								part.Color = bodyColors.LeftLegColor3
							else
								part.Color = bodyColors.RightLegColor3
							end
						elseif part.Name:find("Torso") then
							part.Color = bodyColors.TorsoColor3
						end
					end
				end
			end
		end
	end)
	if not success8 then
		warn("Error applying skin tone:", err8)
	end
	
	if tempModel then
		pcall(function()
			tempModel:Destroy()
		end)
	end
	
	pcall(function()
		avatarModel:Destroy()
	end)

	-- Mark operation as successful
	operationSuccessful = true

	return true, nil
end

-- Copy avatar to a player
-- Check if character is fully loaded and ready
local function isCharacterReady(char, humanoid)
	if not char or not humanoid then return false end

	-- Check if humanoid is properly initialized
	if humanoid.Health <= 0 then return false end

	-- Check if basic body parts exist
	local requiredParts = {"Head", "HumanoidRootPart", "Torso"}
	for _, partName in ipairs(requiredParts) do
		local part = char:FindFirstChild(partName)
		if not part then return false end
	end

	-- For R15, check for additional parts
	if humanoid.RigType == Enum.HumanoidRigType.R15 then
		local r15Parts = {"UpperTorso", "LowerTorso", "LeftUpperArm", "RightUpperArm", "LeftUpperLeg", "RightUpperLeg"}
		for _, partName in ipairs(r15Parts) do
			local part = char:FindFirstChild(partName)
			if not part then return false end
		end
	end

	-- Check if character has been alive for at least a moment
	local aliveTime = tick() - (humanoid:GetAttribute("SpawnTime") or 0)
	if aliveTime < 0.5 then return false end -- Wait at least 0.5 seconds after spawn

	return true
end

local function copyAvatarToPlayer(fromUserId, toPlayer)
	local char = toPlayer.Character
	if not char then
		local errorMsg = "Target player has no character"
		warn(errorMsg)
		return false, errorMsg
	end

	local humanoid = getHumanoid(char)
	if not humanoid then
		local errorMsg = "Target character has no Humanoid"
		warn(errorMsg)
		return false, errorMsg
	end

	-- Mark spawn time for character readiness checking
	if not humanoid:GetAttribute("SpawnTime") then
		humanoid:SetAttribute("SpawnTime", tick())
	end

	-- Wait for character to be fully ready before proceeding
	local readyStartTime = tick()
	while not isCharacterReady(char, humanoid) and (tick() - readyStartTime) < 5 do
		task.wait(0.1)
		log(LOG_LEVEL.DEBUG, "Waiting for character to be ready... (%.1fs)", tick() - readyStartTime)
	end

	if not isCharacterReady(char, humanoid) then
		local errorMsg = "Character failed to initialize properly within 5 seconds"
		log(LOG_LEVEL.ERROR, errorMsg)
		return false, errorMsg
	end

	log(LOG_LEVEL.DEBUG, "Character ready for avatar application after %.2fs", tick() - readyStartTime)

	-- Apply extra delay if enabled
	if extraDelayEnabled then
		log(LOG_LEVEL.DEBUG, "Applying extra 2-second delay...")
		log(LOG_LEVEL.INFO, "Extra delay active - waiting 2 seconds...")
		task.wait(2)
	end

	-- Update UI to show character is ready
	log(LOG_LEVEL.INFO, "Character ready - applying avatar...")

	-- Store original player data before modifying
	storeOriginalPlayerData(toPlayer)

	-- Apply avatar to character with better error handling
	log(LOG_LEVEL.INFO, "Applying avatar from UserId %d to player %s (Safe Mode: %s)", fromUserId, toPlayer.Name, safeModeEnabled and "ON" or "OFF")
	if safeModeEnabled then
		log(LOG_LEVEL.INFO, "Safe mode active - skipping body parts, animations, and scaling")
	end
	local success, errorMsg = applyAvatarToCharacter(fromUserId, char, humanoid)
	if not success then
		log(LOG_LEVEL.ERROR, "Failed to apply avatar to character: %s", errorMsg or "Unknown error")

		-- If initial application fails, don't set up respawn handling to prevent corruption
		toPlayer:SetAttribute("AvatarCopied", false)
		return false, errorMsg or "Avatar application failed - character may be corrupted"
	end

	log(LOG_LEVEL.INFO, "Avatar applied successfully to %s", toPlayer.Name)

	-- Get source player's username and display name
	local sourceUsername, sourceDisplayName = getPlayerInfo(fromUserId)
	
	-- CRITICAL CHECK: If we couldn't get the source username, warn but continue with avatar only
	if not sourceUsername or sourceUsername == "" then
		warn("=== WARNING: Could not fetch source player name ===")
		warn("UserId:", fromUserId, "returned no username")
		warn("Avatar was applied but name change will NOT work")
		warn("This is usually due to Roblox API limitations in executors")
		warn("==============================================")
		
		-- Still mark as copied so avatar persists on respawn
		toPlayer:SetAttribute("SourceUserId", fromUserId)
		toPlayer:SetAttribute("AvatarCopied", true)
		return true, "Avatar applied but name change failed - API returned nil"
	end
	
	-- Use username as displayName if displayName is nil
	if not sourceDisplayName or sourceDisplayName == "" then
		sourceDisplayName = sourceUsername
		log(LOG_LEVEL.DEBUG, "Using username as displayName since displayName was nil")
	end
	
	log(LOG_LEVEL.DEBUG, "Source Player: %s (%s)", sourceUsername, sourceDisplayName)
	
	-- Set CharacterAppearanceId to SOURCE's userId
	local success1 = setProp(toPlayer, "CharacterAppearanceId", fromUserId)
	log(LOG_LEVEL.DEBUG, "Set CharacterAppearanceId: %s", success1 and "SUCCESS" or "FAILED")
	
	-- Set userId to SOURCE's userId (lowercase)
	local success2 = setProp(toPlayer, "userId", fromUserId)
	log(LOG_LEVEL.DEBUG, "Set userId: %s", success2 and "SUCCESS" or "FAILED")
	
	-- Set UserId to SOURCE's userId (uppercase - alias)
	local success3 = setProp(toPlayer, "UserId", fromUserId)
	log(LOG_LEVEL.DEBUG, "Set UserId: %s", success3 and "SUCCESS" or "FAILED")
	
	-- Set Name to SOURCE's USERNAME (not display name!)
	local success4 = setProp(toPlayer, "Name", sourceUsername)
	log(LOG_LEVEL.DEBUG, "Set Name to USERNAME: %s -> %s", success4 and "SUCCESS" or "FAILED", sourceUsername)
	
	-- Set DisplayName to SOURCE's DISPLAY NAME
	local success5 = setProp(toPlayer, "DisplayName", sourceDisplayName)
	log(LOG_LEVEL.DEBUG, "Set DisplayName: %s -> %s", success5 and "SUCCESS" or "FAILED", sourceDisplayName)
	
	-- Set CharacterAppearance URL
	local appearanceUrl = getCharacterAppearanceUrl(fromUserId)
	local success6 = setProp(toPlayer, "CharacterAppearance", appearanceUrl)
	log(LOG_LEVEL.DEBUG, "Set CharacterAppearance: %s", success6 and "SUCCESS" or "FAILED")
	
	-- Rename character model to source username
	pcall(function()
		char.Name = sourceUsername
	end)
	log(LOG_LEVEL.DEBUG, "Set Character.Name: %s", sourceUsername)
	
	-- Store source UserId and mark as active
	toPlayer:SetAttribute("SourceUserId", fromUserId)
	toPlayer:SetAttribute("AvatarCopied", true)
	
	-- Set up respawn connection FIRST before setting display name
	-- This ensures the avatar persists after respawn
	if not activeConnections[toPlayer.UserId] then
		activeConnections[toPlayer.UserId] = {}
	end
	
	-- Set up CharacterAdded connection for respawn persistence
	local respawnConnection = toPlayer.CharacterAdded:Connect(function(newChar)
		log(LOG_LEVEL.DEBUG, "CharacterAdded fired for player %s", toPlayer.Name)

		-- Check if we should still be applying avatars (user might have reverted)
		if not toPlayer:GetAttribute("AvatarCopied") then
			log(LOG_LEVEL.DEBUG, "Avatar copying disabled for %s, skipping respawn handling", toPlayer.Name)
			return
		end

		-- Check if auto-respawn is enabled
		if not autoRespawnEnabled then
			log(LOG_LEVEL.DEBUG, "Auto-respawn disabled, skipping avatar reapplication for %s", toPlayer.Name)
			return
		end

		-- Wait for character to fully load with multiple checks
		local newHumanoid = nil
		for i = 1, 15 do
			newHumanoid = newChar:FindFirstChildOfClass("Humanoid")
			if newHumanoid then break end
			task.wait(0.1)
		end

		if not newHumanoid then
			log(LOG_LEVEL.WARN, "Could not find Humanoid on respawn for %s", toPlayer.Name)
			return
		end

		-- Wait for Roblox to fully apply the default avatar first
		-- This is crucial - we need to wait for body parts, clothing, etc. to load
		task.wait(1.0)

		-- Additional wait to ensure all body parts exist
		local function waitForBodyParts()
			local requiredParts
			if newHumanoid.RigType == Enum.HumanoidRigType.R15 then
				requiredParts = {"Head", "UpperTorso", "LowerTorso", "LeftUpperArm", "RightUpperArm", "LeftUpperLeg", "RightUpperLeg"}
			else
				requiredParts = {"Head", "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg"}
			end
			
			for i = 1, 20 do
				local allFound = true
				for _, partName in ipairs(requiredParts) do
					if not newChar:FindFirstChild(partName) then
						allFound = false
						break
					end
				end
				if allFound then return true end
				task.wait(0.1)
			end
			return false
		end
		
		if not waitForBodyParts() then
			log(LOG_LEVEL.WARN, "Body parts not fully loaded for %s, proceeding anyway", toPlayer.Name)
		end

		-- One more wait to let everything settle
		task.wait(0.5)

		local storedSourceUserId = toPlayer:GetAttribute("SourceUserId")
		if storedSourceUserId and toPlayer:GetAttribute("AvatarCopied") and autoRespawnEnabled then
			log(LOG_LEVEL.DEBUG, "Reapplying avatar on respawn for player %s", toPlayer.Name)

			-- Try to apply avatar with better error handling
			local applySuccess = false
			local lastError = ""

			for attempt = 1, 5 do
				-- Re-check character validity before each attempt
				if toPlayer.Character ~= newChar then
					log(LOG_LEVEL.WARN, "Character changed during respawn handling, aborting")
					return
				end

				local success2, result2 = pcall(function()
					-- Apply avatar normally without forcing safe mode
					local result = applyAvatarToCharacter(storedSourceUserId, newChar, newHumanoid, true)
					return result
				end)

				if success2 and result2 then
					applySuccess = true
					log(LOG_LEVEL.DEBUG, "Successfully reapplied avatar on attempt %d", attempt)
					break
				else
					lastError = tostring(result2 or "Unknown error")
					log(LOG_LEVEL.WARN, "Avatar reapplication failed on attempt %d: %s", attempt, lastError)
					-- Exponential backoff with longer waits
					task.wait(0.3 * attempt)
				end
			end

			if not applySuccess then
				log(LOG_LEVEL.ERROR, "Failed to reapply avatar on respawn after 5 attempts: %s", lastError)
				-- If avatar reapplication keeps failing, disable it to prevent character corruption
				local respawnFailureCount = toPlayer:GetAttribute("RespawnFailureCount") or 0
				respawnFailureCount = respawnFailureCount + 1
				toPlayer:SetAttribute("RespawnFailureCount", respawnFailureCount)

				if respawnFailureCount >= 3 then
					log(LOG_LEVEL.ERROR, "Avatar reapplication failed 3+ times, disabling to prevent character corruption")
					toPlayer:SetAttribute("AvatarCopied", false)  -- Disable further attempts
				end
			else
				-- Reset failure count on success
				toPlayer:SetAttribute("RespawnFailureCount", 0)
			end
			
			-- Reapply display name
			local sourceUsername2, sourceDisplayName2 = getPlayerInfo(storedSourceUserId)
			local displayNameToSet = sourceDisplayName2 or sourceUsername2
			if displayNameToSet then
				-- Try multiple times
				for i = 1, 3 do
					local success = pcall(function()
						newHumanoid.DisplayName = displayNameToSet
					end)
					if success then break end
					task.wait(0.1)
				end
			end
			
			-- Re-setup name monitoring (the old connections are still active for UI)
			if sourceUsername2 then
				-- Update existing name tag if present
				local headPart = newChar:FindFirstChild("Head")
				if headPart then
					for _, child in ipairs(headPart:GetChildren()) do
						if child:IsA("BillboardGui") then
							local textLabel = child:FindFirstChild("TextLabel")
							if textLabel then
								textLabel.Text = displayNameToSet
							end
						end
					end
				end
			end
		end
	end)
	table.insert(activeConnections[toPlayer.UserId], respawnConnection)
	
	-- Set display name on Humanoid (this shows above the character's head)
	-- ALWAYS prioritize display name over username
	if humanoid then
		local displayNameToSet = nil
		
		-- First try to get the actual display name from the source player
		if sourceDisplayName and sourceDisplayName ~= "" then
			displayNameToSet = sourceDisplayName
		elseif sourceUsername and sourceUsername ~= "" then
			displayNameToSet = sourceUsername
		end
		
		if displayNameToSet then
			-- Set it immediately
			local success = pcall(function()
				humanoid.DisplayName = displayNameToSet
			end)
			
			if not success then
				-- Try again after a small delay
				task.spawn(function()
					task.wait(0.1)
					pcall(function()
						if humanoid and humanoid.Parent then
							humanoid.DisplayName = displayNameToSet
						end
					end)
				end)
			end
		end
	end
	
	-- Change the target player's name to match source
	if sourceUsername then
		changePlayerName(toPlayer, sourceUsername, sourceDisplayName, fromUserId)
	end

	-- Profile and player list interception is handled by changePlayerName monitoring system

	log(LOG_LEVEL.INFO, "Copied avatar %d > %s", fromUserId, toPlayer.Name)
	return true, nil
end


-- Get target player from input
local function getTargetPlayer(input)
	input = tostring(input or ""):gsub("^%s+", ""):gsub("%s+$", "")
	
	if input == "" or input:lower() == "me" or input:lower() == "self" then
		return LocalPlayer
	end
	
	-- Try as User ID first
	local userId = tonumber(input)
	if userId then
		for _, plr in ipairs(Players:GetPlayers()) do
			if plr.UserId == userId then
				return plr
			end
		end
	end
	
	-- Try as username
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr.Name == input or plr.DisplayName == input then
			return plr
		end
	end
	
	return nil
end

-- Revert function
local function revertPlayer(toPlayer)
	if not toPlayer then
		warn("Cannot revert: player is nil")
		return false
	end
	
	local char = toPlayer.Character
	if not char then
		warn("Cannot revert: player has no character")
		return false
	end
	
	local humanoid = getHumanoid(char)
	if not humanoid then
		warn("Cannot revert: character has no Humanoid")
		return false
	end
	
	-- Get original data
	local original = originalPlayerData[toPlayer.UserId]
	local originalUserId = original and original.odUserId or toPlayer.UserId
	
	-- ============================================
	-- RESTORE PLAYER PROPERTIES
	-- ============================================
	
	log(LOG_LEVEL.DEBUG, "Restoring player properties...")
	
	if original then
		-- Restore CharacterAppearanceId
		setProp(toPlayer, "CharacterAppearanceId", original.characterAppearanceId or originalUserId)
		print("Restored CharacterAppearanceId")
		
		-- Restore userId
		setProp(toPlayer, "userId", originalUserId)
		print("Restored userId")
		
		-- Restore UserId
		setProp(toPlayer, "UserId", originalUserId)
		print("Restored UserId")
		
		-- Restore Name
		setProp(toPlayer, "Name", original.name)
		print("Restored Name:", original.name)
		
		-- Restore DisplayName
		setProp(toPlayer, "DisplayName", original.displayName)
		print("Restored DisplayName:", original.displayName)
		
		-- Restore CharacterAppearance
		setProp(toPlayer, "CharacterAppearance", getCharacterAppearanceUrl(originalUserId))
		print("Restored CharacterAppearance")
		
		-- Restore character model name
		pcall(function()
			char.Name = original.name
		end)
		
		-- Restore humanoid display name
		pcall(function()
			humanoid.DisplayName = original.humanoidDisplayName or original.displayName
		end)
	end
	

	
	-- Restore original avatar
	local avatarModel = getAvatarModel(originalUserId)
	if avatarModel then
		clearCosmetics(char)
		copyClothingAndColors(avatarModel, char)
		copyBodyMeshes(avatarModel, char, humanoid)  -- Pass humanoid for R15 body parts
		copyFace(originalUserId, char, avatarModel)  -- Restore original face
		copyAccessories(avatarModel, humanoid, char, originalUserId)
		avatarModel:Destroy()
	end
	
	-- Stop monitoring
	toPlayer:SetAttribute("MonitoringActive", false)
	toPlayer:SetAttribute("AvatarCopied", false)
	
	-- Clear fake name attributes
	toPlayer:SetAttribute("FakeUsername", nil)
	toPlayer:SetAttribute("FakeDisplayName", nil)
	toPlayer:SetAttribute("SourceUserId", nil)
	
	-- Disconnect monitoring connections
	if activeConnections[toPlayer.UserId] then
		for _, connection in ipairs(activeConnections[toPlayer.UserId]) do
			if connection and connection.Disconnect then
				connection:Disconnect()
			end
		end
		activeConnections[toPlayer.UserId] = nil
	end
	
	-- Clear original data
	originalPlayerData[toPlayer.UserId] = nil
	
	print("Reverted avatar and names for", toPlayer.Name)
	return true
end

-- Create UI
local function createGUI()
	local pg = LocalPlayer:WaitForChild("PlayerGui")
	local existing = pg:FindFirstChild("AvatarCopierGUI")
	if existing then existing:Destroy() end

	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "AvatarCopierGUI"
	screenGui.ResetOnSpawn = false
	screenGui.Parent = pg

	local frame = Instance.new("Frame")
	frame.Name = "Main"
	frame.Size = UDim2.new(0, 420, 0, 620)
	frame.Position = UDim2.new(0.5, -210, 0.5, -190)
	frame.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
	frame.BorderSizePixel = 0
	frame.ClipsDescendants = true  -- This ensures content is hidden when frame is minimized
	frame.Parent = screenGui

	-- Main gradient background
	local mainGradient = Instance.new("UIGradient")
	mainGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(18, 18, 24)),
		ColorSequenceKeypoint.new(0.7, Color3.fromRGB(22, 22, 30)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(16, 16, 22))
	}
	mainGradient.Rotation = 90
	mainGradient.Parent = frame

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 14)
	corner.Parent = frame

	-- Subtle stroke
	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(45, 45, 60)
	stroke.Thickness = 1
	stroke.Transparency = 0.8
	stroke.Parent = frame

	-- Topbar (taller to fit tabs)
	local topbar = Instance.new("Frame")
	topbar.Name = "Topbar"
	topbar.Size = UDim2.new(1, 0, 0, 50)
	topbar.BackgroundColor3 = Color3.fromRGB(28, 28, 36)
	topbar.BorderSizePixel = 0
	topbar.Parent = frame

	local topGradient = Instance.new("UIGradient")
	topGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(32, 32, 42)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(24, 24, 32))
	}
	topGradient.Rotation = 90
	topGradient.Parent = topbar

	local topCorner = Instance.new("UICorner")
	topCorner.CornerRadius = UDim.new(0, 14)
	topCorner.Parent = topbar

	local title = Instance.new("TextLabel")
	title.BackgroundTransparency = 1
	title.Position = UDim2.new(0, 12, 0, 4)
	title.Size = UDim2.new(1, -80, 0, 14)
	title.Text = "Heil Holy Grail"
	title.TextColor3 = Color3.fromRGB(240, 240, 250)
	title.Font = Enum.Font.GothamBold
	title.TextSize = 12
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = topbar

	local subtitle = Instance.new("TextLabel")
	subtitle.BackgroundTransparency = 1
	subtitle.Position = UDim2.new(0, 12, 0, 17)
	subtitle.Size = UDim2.new(1, -80, 0, 12)
	subtitle.Text = "Is this.. Holy Grail?"
	subtitle.TextColor3 = Color3.fromRGB(160, 160, 180)
	subtitle.Font = Enum.Font.GothamMedium
	subtitle.TextSize = 9
	subtitle.TextXAlignment = Enum.TextXAlignment.Left
	subtitle.Parent = topbar

	local function makeTopButton(text, xOffset, bgColor, hoverColor)
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(0, 26, 0, 22)
		btn.Position = UDim2.new(1, xOffset, 0, 4)
		btn.BackgroundColor3 = bgColor or Color3.fromRGB(45, 45, 55)
		btn.TextColor3 = Color3.fromRGB(220, 220, 230)
		btn.Font = Enum.Font.GothamBold
		btn.TextSize = 16
		btn.Text = text
		btn.AutoButtonColor = false
		btn.Parent = topbar

		local btnCorner = Instance.new("UICorner")
		btnCorner.CornerRadius = UDim.new(0, 8)
		btnCorner.Parent = btn

		local btnStroke = Instance.new("UIStroke")
		btnStroke.Color = Color3.fromRGB(65, 65, 80)
		btnStroke.Thickness = 1
		btnStroke.Transparency = 0.7
		btnStroke.Parent = btn

		-- Hover effects
		local originalColor = btn.BackgroundColor3
		local targetColor = hoverColor or Color3.fromRGB(55, 55, 70)

		btn.MouseEnter:Connect(function()
			local tween = game:GetService("TweenService"):Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quart), {BackgroundColor3 = targetColor})
			tween:Play()
		end)

		btn.MouseLeave:Connect(function()
			local tween = game:GetService("TweenService"):Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quart), {BackgroundColor3 = originalColor})
			tween:Play()
		end)

		-- Add UIGradient for better visual effect
		local btnGradient = Instance.new("UIGradient")
		btnGradient.Color = ColorSequence.new{
			ColorSequenceKeypoint.new(0, Color3.fromRGB(75, 120, 240)),
			ColorSequenceKeypoint.new(0.5, bgColor or Color3.fromRGB(45, 45, 55)),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(45, 80, 200))
		}
		btnGradient.Parent = btn

		return btn
	end

	local minimizeBtn = makeTopButton("-", -62, Color3.fromRGB(25, 60, 180), Color3.fromRGB(50, 90, 220))
	-- Add theme gradient to minimize button
	local minimizeGradient = minimizeBtn:FindFirstChildOfClass("UIGradient")
	if minimizeGradient then
		minimizeGradient.Color = ColorSequence.new{
			ColorSequenceKeypoint.new(0, Color3.fromRGB(60, 100, 230)),
			ColorSequenceKeypoint.new(0.5, Color3.fromRGB(35, 75, 200)),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(25, 60, 180))
		}
	end
	local closeBtn = makeTopButton("Ã—", -32, Color3.fromRGB(75, 35, 35), Color3.fromRGB(95, 45, 45))

	-- Override gradient for close button with revert avatar theme (red)
	local closeGradient = closeBtn:FindFirstChildOfClass("UIGradient")
	if closeGradient then closeGradient:Destroy() end
	closeGradient = Instance.new("UIGradient")
	closeGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(200, 70, 70)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(180, 50, 50)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(160, 30, 30))
	}
	closeGradient.Parent = closeBtn

	-- Theme color (can be changed by user)
	local themeColor = Color3.fromRGB(25, 60, 180)
	local themeColorLight = Color3.fromRGB(35, 70, 200)
	local themeColorDark = Color3.fromRGB(20, 50, 160)

	-- Tab buttons container (positioned next to minimize button) with nice styling
	local tabContainer = Instance.new("Frame")
	tabContainer.Name = "TabContainer"
	tabContainer.Size = UDim2.new(0, 200, 0, 28)
	tabContainer.Position = UDim2.new(1, -280, 0, 4)
	tabContainer.BackgroundColor3 = Color3.fromRGB(10, 10, 14)
	tabContainer.Parent = topbar

	local tabContainerCorner = Instance.new("UICorner")
	tabContainerCorner.CornerRadius = UDim.new(0, 10)
	tabContainerCorner.Parent = tabContainer

	local tabContainerStroke = Instance.new("UIStroke")
	tabContainerStroke.Color = Color3.fromRGB(30, 30, 40)
	tabContainerStroke.Thickness = 1
	tabContainerStroke.Transparency = 0.3
	tabContainerStroke.Parent = tabContainer

	local tabContainerPad = Instance.new("UIPadding")
	tabContainerPad.PaddingLeft = UDim.new(0, 3)
	tabContainerPad.PaddingRight = UDim.new(0, 3)
	tabContainerPad.PaddingTop = UDim.new(0, 3)
	tabContainerPad.PaddingBottom = UDim.new(0, 3)
	tabContainerPad.Parent = tabContainer

	local tabLayout = Instance.new("UIListLayout")
	tabLayout.FillDirection = Enum.FillDirection.Horizontal
	tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
	tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
	tabLayout.Padding = UDim.new(0, 3)
	tabLayout.Parent = tabContainer

	-- Tab button creator
	local currentTab = "frame"
	local tabButtons = {}
	local tabGradients = {}
	local tabContents = {}

	-- Gradient colors for selected/unselected states
	local function getSelectedGradient(color)
		return ColorSequence.new{
			ColorSequenceKeypoint.new(0, Color3.fromRGB(
				math.min(255, color.R * 255 + 30),
				math.min(255, color.G * 255 + 40),
				math.min(255, color.B * 255 + 50)
			)),
			ColorSequenceKeypoint.new(0.5, color),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(
				math.max(0, color.R * 255 - 10),
				math.max(0, color.G * 255 - 15),
				math.max(0, color.B * 255 - 20)
			))
		}
	end

	local unselectedGradient = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(22, 22, 30)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(18, 18, 24)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(14, 14, 18))
	}

	local function makeTabButton(name, displayName, order)
		local btn = Instance.new("TextButton")
		btn.Name = name .. "Tab"
		btn.LayoutOrder = order
		btn.Size = UDim2.new(0, 58, 0, 24)
		btn.BackgroundColor3 = name == "frame" and themeColor or Color3.fromRGB(18, 18, 24)
		btn.TextColor3 = Color3.fromRGB(220, 220, 230)
		btn.Font = Enum.Font.GothamBold
		btn.TextSize = 11
		btn.Text = displayName
		btn.AutoButtonColor = false
		btn.Parent = tabContainer

		local btnCorner = Instance.new("UICorner")
		btnCorner.CornerRadius = UDim.new(0, 6)
		btnCorner.Parent = btn

		local btnGradient = Instance.new("UIGradient")
		btnGradient.Rotation = 90
		if name == "frame" then
			btnGradient.Color = getSelectedGradient(themeColor)
		else
			btnGradient.Color = unselectedGradient
		end
		btnGradient.Parent = btn

		tabButtons[name] = btn
		tabGradients[name] = btnGradient
		return btn
	end

	local frameTabBtn = makeTabButton("frame", "Frame", 1)
	local selfTabBtn = makeTabButton("self", "Self", 2)
	local miscTabBtn = makeTabButton("misc", "Misc", 3)

	-- Function to switch tabs
	local function switchTab(tabName)
		currentTab = tabName
		for name, btn in pairs(tabButtons) do
			if name == tabName then
				btn.BackgroundColor3 = themeColor
				if tabGradients[name] then
					tabGradients[name].Color = getSelectedGradient(themeColor)
				end
			else
				btn.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
				if tabGradients[name] then
					tabGradients[name].Color = unselectedGradient
				end
			end
		end
		for name, content in pairs(tabContents) do
			content.Visible = (name == tabName)
		end
	end

	frameTabBtn.MouseButton1Click:Connect(function() switchTab("frame") end)
	selfTabBtn.MouseButton1Click:Connect(function() switchTab("self") end)
	miscTabBtn.MouseButton1Click:Connect(function() switchTab("misc") end)

	-- Body container
	local body = Instance.new("Frame")
	body.Name = "Body"
	body.BackgroundTransparency = 1
	body.Position = UDim2.new(0, 0, 0, 50)
	body.Size = UDim2.new(1, 0, 1, -50)
	body.ClipsDescendants = true
	body.Parent = frame

	-- Create tab content containers
	local function createTabContent(name)
		local content = Instance.new("ScrollingFrame")
		content.Name = name .. "Content"
		content.Size = UDim2.new(1, 0, 1, 0)
		content.BackgroundTransparency = 1
		content.BorderSizePixel = 0
		content.ScrollBarThickness = 4
		content.ScrollBarImageColor3 = themeColor
		content.CanvasSize = UDim2.new(0, 0, 0, 0)
		content.AutomaticCanvasSize = Enum.AutomaticSize.Y
		content.Visible = (name == "frame")
		content.Parent = body

		local pad = Instance.new("UIPadding")
		pad.PaddingLeft = UDim.new(0, 18)
		pad.PaddingRight = UDim.new(0, 18)
		pad.PaddingTop = UDim.new(0, 12)
		pad.PaddingBottom = UDim.new(0, 18)
		pad.Parent = content

		local layout = Instance.new("UIListLayout")
		layout.Padding = UDim.new(0, 10)
		layout.SortOrder = Enum.SortOrder.LayoutOrder
		layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
		layout.Parent = content

		tabContents[name] = content
		return content
	end

	local frameContent = createTabContent("frame")
	local selfContent = createTabContent("self")
	local miscContent = createTabContent("misc")

	-- Executor status indicator
	local execStatus = Instance.new("TextLabel")
	execStatus.LayoutOrder = 0
	execStatus.BackgroundTransparency = 1
	execStatus.Size = UDim2.new(1, 0, 0, 16)
	execStatus.Text = hasExecutor and "Executor: sethiddenproperty available" or "âš  No executor - Player properties won't change"
	execStatus.TextColor3 = hasExecutor and themeColor or Color3.fromRGB(255, 180, 80)
	execStatus.Font = Enum.Font.GothamMedium
	execStatus.TextSize = 11
	execStatus.TextXAlignment = Enum.TextXAlignment.Left
	execStatus.Parent = frameContent

	-- Status bar
	local statusFrame = Instance.new("Frame")
	statusFrame.LayoutOrder = 1
	statusFrame.Size = UDim2.new(1, 0, 0, 28)
	statusFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 38)
	statusFrame.BorderSizePixel = 0
	statusFrame.Parent = frameContent

	local statusGradient = Instance.new("UIGradient")
	statusGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(32, 32, 45)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(24, 24, 35))
	}
	statusGradient.Rotation = 90
	statusGradient.Parent = statusFrame

	local statusCorner = Instance.new("UICorner")
	statusCorner.CornerRadius = UDim.new(0, 10)
	statusCorner.Parent = statusFrame

	local statusStroke = Instance.new("UIStroke")
	statusStroke.Color = Color3.fromRGB(55, 55, 75)
	statusStroke.Thickness = 1
	statusStroke.Transparency = 0.8
	statusStroke.Parent = statusFrame

	local status = Instance.new("TextLabel")
	status.BackgroundTransparency = 1
	status.Size = UDim2.new(1, -16, 1, 0)
	status.Position = UDim2.new(0, 12, 0, 0)
	status.Text = "Ready..."
	status.TextColor3 = Color3.fromRGB(25, 60, 180)
	status.Font = Enum.Font.GothamMedium
	status.TextSize = 12
	status.TextXAlignment = Enum.TextXAlignment.Left
	status.Parent = statusFrame

	-- Safety toggle for respawn handling
	local respawnToggle = Instance.new("TextButton")
	respawnToggle.Size = UDim2.new(0, 20, 0, 20)
	respawnToggle.Position = UDim2.new(1, -25, 0, 2)
	respawnToggle.BackgroundColor3 = Color3.fromRGB(25,  60,  180)
	respawnToggle.Text = "âœ“"
	respawnToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
	respawnToggle.Font = Enum.Font.GothamBold
	respawnToggle.TextSize = 14
	respawnToggle.Parent = statusFrame

	-- Add UIGradient
	local respawnGradient = Instance.new("UIGradient")
	respawnGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(70, 120, 230)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(45, 90, 210)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 70, 190))
	}
	respawnGradient.Parent = respawnToggle

	local toggleCorner = Instance.new("UICorner")
	toggleCorner.CornerRadius = UDim.new(0, 4)
	toggleCorner.Parent = respawnToggle

	-- Tooltip for the toggle
	local tooltip = Instance.new("TextLabel")
	tooltip.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	tooltip.TextColor3 = Color3.fromRGB(220, 220, 230)
	tooltip.Text = "Auto-respawn\nClick to disable if character doesn't respawn"
	tooltip.Font = Enum.Font.GothamMedium
	tooltip.TextSize = 10
	tooltip.Size = UDim2.new(0, 180, 0, 30)
	tooltip.Position = UDim2.new(0, -185, 0, -35)
	tooltip.Visible = false
	tooltip.ZIndex = 10
	tooltip.Parent = respawnToggle

	local tooltipCorner = Instance.new("UICorner")
	tooltipCorner.CornerRadius = UDim.new(0, 6)
	tooltipCorner.Parent = tooltip

	local tooltipStroke = Instance.new("UIStroke")
	tooltipStroke.Color = Color3.fromRGB(80, 80, 100)
	tooltipStroke.Thickness = 1
	tooltipStroke.Parent = tooltip

	respawnToggle.MouseEnter:Connect(function()
		tooltip.Visible = true
	end)

	respawnToggle.MouseLeave:Connect(function()
		tooltip.Visible = false
	end)

	-- Function to update toggle button appearance
	local function updateToggleAppearance(btn, gradient, isOn)
		if isOn then
			btn.BackgroundColor3 = themeColor
			gradient.Color = ColorSequence.new{
				ColorSequenceKeypoint.new(0, Color3.fromRGB(
					math.min(255, themeColor.R * 255 + 45),
					math.min(255, themeColor.G * 255 + 60),
					math.min(255, themeColor.B * 255 + 50)
				)),
				ColorSequenceKeypoint.new(0.5, Color3.fromRGB(
					math.min(255, themeColor.R * 255 + 20),
					math.min(255, themeColor.G * 255 + 30),
					math.min(255, themeColor.B * 255 + 30)
				)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(
					math.min(255, themeColor.R * 255 + 5),
					math.min(255, themeColor.G * 255 + 10),
					math.min(255, themeColor.B * 255 + 10)
				))
			}
		else
			btn.BackgroundColor3 = Color3.fromRGB(85, 30, 30)
			gradient.Color = ColorSequence.new{
				ColorSequenceKeypoint.new(0, Color3.fromRGB(110, 45, 45)),
				ColorSequenceKeypoint.new(0.5, Color3.fromRGB(85, 35, 35)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 28, 28))
			}
		end
	end

	respawnToggle.MouseButton1Click:Connect(function()
		autoRespawnEnabled = not autoRespawnEnabled
		updateToggleAppearance(respawnToggle, respawnGradient, autoRespawnEnabled)
		if autoRespawnEnabled then
			respawnToggle.Text = "âœ“"
			setStatus("Auto-respawn: ENABLED", Color3.fromRGB(100, 255, 100), 2)
		else
			respawnToggle.Text = "X"
			setStatus("Auto-respawn: DISABLED (use for games with respawn issues)", Color3.fromRGB(255, 200, 100), 3)
		end
	end)

	-- Safe Mode Toggle (starts OFF = red)
	local safeModeToggle = Instance.new("TextButton")
	safeModeToggle.Size = UDim2.new(0, 20, 0, 20)
	safeModeToggle.Position = UDim2.new(1, -50, 0, 2)
	safeModeToggle.BackgroundColor3 = Color3.fromRGB(85, 30, 30)
	safeModeToggle.Text = "ðŸ›¡ï¸"
	safeModeToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
	safeModeToggle.Font = Enum.Font.GothamBold
	safeModeToggle.TextSize = 12
	safeModeToggle.Parent = statusFrame

	-- Add UIGradient (starts red = OFF)
	local safeGradient = Instance.new("UIGradient")
	safeGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(110, 45, 45)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(85, 35, 35)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 28, 28))
	}
	safeGradient.Parent = safeModeToggle

	local safeCorner = Instance.new("UICorner")
	safeCorner.CornerRadius = UDim.new(0, 4)
	safeCorner.Parent = safeModeToggle

	local safeTooltip = Instance.new("TextLabel")
	safeTooltip.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	safeTooltip.TextColor3 = Color3.fromRGB(220, 220, 230)
	safeTooltip.Text = "Safe Mode\nClick to enable safe avatar copying\n(skips body parts & animations)"
	safeTooltip.Font = Enum.Font.GothamMedium
	safeTooltip.TextSize = 10
	safeTooltip.Size = UDim2.new(0, 180, 0, 40)
	safeTooltip.Position = UDim2.new(0, -210, 0, -45)
	safeTooltip.Visible = false
	safeTooltip.ZIndex = 10
	safeTooltip.Parent = safeModeToggle

	local safeStroke = Instance.new("UIStroke")
	safeStroke.Color = Color3.fromRGB(80, 80, 100)
	safeStroke.Thickness = 1
	safeStroke.Parent = safeTooltip

	local safeCorner2 = Instance.new("UICorner")
	safeCorner2.CornerRadius = UDim.new(0, 6)
	safeCorner2.Parent = safeTooltip

	safeModeToggle.MouseEnter:Connect(function()
		safeTooltip.Visible = true
	end)

	safeModeToggle.MouseLeave:Connect(function()
		safeTooltip.Visible = false
	end)

	safeModeToggle.MouseButton1Click:Connect(function()
		safeModeEnabled = not safeModeEnabled
		updateToggleAppearance(safeModeToggle, safeGradient, safeModeEnabled)
		safeModeToggle.Text = "ðŸ›¡ï¸"
		if safeModeEnabled then
			setStatus("Safe Mode: ENABLED (body parts & animations skipped)", Color3.fromRGB(100, 200, 200), 3)
		else
			setStatus("Safe Mode: DISABLED", Color3.fromRGB(200, 200, 100), 2)
		end
	end)

	-- Manual Delay Toggle (starts OFF = red)
	local delayToggle = Instance.new("TextButton")
	delayToggle.Size = UDim2.new(0, 20, 0, 20)
	delayToggle.Position = UDim2.new(1, -75, 0, 2)
	delayToggle.BackgroundColor3 = Color3.fromRGB(85, 30, 30)
	delayToggle.Text = "â±ï¸"
	delayToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
	delayToggle.Font = Enum.Font.GothamBold
	delayToggle.TextSize = 12
	delayToggle.Parent = statusFrame

	-- Add UIGradient (starts red = OFF)
	local delayGradient = Instance.new("UIGradient")
	delayGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(110, 45, 45)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(85, 35, 35)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 28, 28))
	}
	delayGradient.Parent = delayToggle

	local delayCorner = Instance.new("UICorner")
	delayCorner.CornerRadius = UDim.new(0, 4)
	delayCorner.Parent = delayToggle

	local delayTooltip = Instance.new("TextLabel")
	delayTooltip.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	delayTooltip.TextColor3 = Color3.fromRGB(220, 220, 230)
	delayTooltip.Text = "Extra Delay\nClick to add 2-second delay\n(before avatar application)"
	delayTooltip.Font = Enum.Font.GothamMedium
	delayTooltip.TextSize = 10
	delayTooltip.Size = UDim2.new(0, 180, 0, 40)
	delayTooltip.Position = UDim2.new(0, -235, 0, -45)
	delayTooltip.Visible = false
	delayTooltip.ZIndex = 10
	delayTooltip.Parent = delayToggle

	local delayStroke = Instance.new("UIStroke")
	delayStroke.Color = Color3.fromRGB(80, 80, 100)
	delayStroke.Thickness = 1
	delayStroke.Parent = delayTooltip

	local delayCorner2 = Instance.new("UICorner")
	delayCorner2.CornerRadius = UDim.new(0, 6)
	delayCorner2.Parent = delayTooltip

	delayToggle.MouseEnter:Connect(function()
		delayTooltip.Visible = true
	end)

	delayToggle.MouseLeave:Connect(function()
		delayTooltip.Visible = false
	end)

	local extraDelayEnabled = false
	delayToggle.MouseButton1Click:Connect(function()
		extraDelayEnabled = not extraDelayEnabled
		updateToggleAppearance(delayToggle, delayGradient, extraDelayEnabled)
		delayToggle.Text = "â±ï¸"
		if extraDelayEnabled then
			setStatus("Extra Delay: ENABLED (+2 seconds)", Color3.fromRGB(200, 100, 200), 3)
		else
			setStatus("Extra Delay: DISABLED", Color3.fromRGB(100, 100, 200), 2)
		end
	end)

	-- Progress bar for loading states
	local progressBar = Instance.new("Frame")
	progressBar.Size = UDim2.new(0, 0, 0, 2)
	progressBar.Position = UDim2.new(0, 12, 0, 20)
	progressBar.BackgroundColor3 = Color3.fromRGB(100, 200, 100)
	progressBar.BorderSizePixel = 0
	progressBar.Parent = statusFrame

	local progressCorner = Instance.new("UICorner")
	progressCorner.CornerRadius = UDim.new(0, 1)
	progressCorner.Parent = progressBar

	local function makeLabel(text, order, icon)
		local lbl = Instance.new("TextLabel")
		lbl.LayoutOrder = order
		lbl.BackgroundTransparency = 1
		lbl.Size = UDim2.new(1, 0, 0, 16)
		lbl.Text = (icon or "ðŸ“‹") .. " " .. text
		lbl.TextColor3 = Color3.fromRGB(200, 200, 210)
		lbl.Font = Enum.Font.GothamSemibold
		lbl.TextSize = 13
		lbl.TextXAlignment = Enum.TextXAlignment.Left
		return lbl
	end

	local function makeBox(placeholder, order, icon)
		local container = Instance.new("Frame")
		container.LayoutOrder = order
		container.Size = UDim2.new(1, 0, 0, 44)
		container.BackgroundTransparency = 1
		container.Name = "InputContainer"

		local box = Instance.new("TextBox")
		box.Size = UDim2.new(1, 0, 1, 0)
		box.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
		box.TextColor3 = Color3.fromRGB(240, 240, 245)
		box.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
		box.PlaceholderText = (icon or "âœï¸") .. " " .. placeholder
		box.Font = Enum.Font.GothamMedium
		box.TextSize = 14
		box.ClearTextOnFocus = false
		box.Text = ""
		box.TextXAlignment = Enum.TextXAlignment.Left
		box.Parent = container

		-- Max 20 characters
		box:GetPropertyChangedSignal("Text"):Connect(function()
			if #box.Text > 20 then
				box.Text = box.Text:sub(1, 20)
			end
		end)

		local boxPad = Instance.new("UIPadding")
		boxPad.PaddingLeft = UDim.new(0, 12)
		boxPad.PaddingRight = UDim.new(0, 12)
		boxPad.Parent = box

		local boxCorner = Instance.new("UICorner")
		boxCorner.CornerRadius = UDim.new(0, 10)
		boxCorner.Parent = box

		local boxStroke = Instance.new("UIStroke")
		boxStroke.Color = Color3.fromRGB(65, 65, 85)
		boxStroke.Thickness = 1
		boxStroke.Transparency = 0.6
		boxStroke.Parent = box

		-- Focus effects
		local originalStrokeColor = boxStroke.Color
		local originalStrokeTransparency = boxStroke.Transparency

		box.Focused:Connect(function()
			local tween = game:GetService("TweenService"):Create(boxStroke, TweenInfo.new(0.2), {
				Transparency = 0.2,
				Color = Color3.fromRGB(80, 120, 200)
			})
			tween:Play()
		end)

		box.FocusLost:Connect(function()
			local tween = game:GetService("TweenService"):Create(boxStroke, TweenInfo.new(0.2), {
				Transparency = originalStrokeTransparency,
				Color = originalStrokeColor
			})
			tween:Play()
		end)

		return box, container
	end

	local srcLabel = makeLabel("Source Player", 2, "â€¢")
	srcLabel.Parent = frameContent

	-- Icons for Frame Tab
	local function createIconBox(placeholder, order, parent)
		local container = Instance.new("Frame")
		container.LayoutOrder = order
		container.Size = UDim2.new(0.48, 0, 0, 44)
		container.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
		container.Parent = parent
		
		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 10)
		corner.Parent = container
		
		local stroke = Instance.new("UIStroke")
		stroke.Color = Color3.fromRGB(65, 65, 85)
		stroke.Thickness = 1
		stroke.Transparency = 0.6
		stroke.Parent = container

		local box = Instance.new("TextBox")
		box.Size = UDim2.new(1, -20, 1, 0)
		box.Position = UDim2.new(0, 10, 0, 0)
		box.BackgroundTransparency = 1
		box.TextColor3 = Color3.fromRGB(240, 240, 245)
		box.PlaceholderText = placeholder
		box.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
		box.Font = Enum.Font.GothamMedium
		box.TextSize = 13
		box.Text = ""
		box.ClearTextOnFocus = false
		box.TextXAlignment = Enum.TextXAlignment.Left
		box.Parent = container
		
		-- Focus animation
		box.Focused:Connect(function()
			game:GetService("TweenService"):Create(stroke, TweenInfo.new(0.2), {Transparency = 0.2, Color = Color3.fromRGB(80, 120, 200)}):Play()
		end)
		box.FocusLost:Connect(function()
			game:GetService("TweenService"):Create(stroke, TweenInfo.new(0.2), {Transparency = 0.6, Color = Color3.fromRGB(65, 65, 85)}):Play()
		end)
		
		return box
	end
	
	local frameIconsContainer = Instance.new("Frame")
	frameIconsContainer.LayoutOrder = 1
	frameIconsContainer.Size = UDim2.new(1, 0, 0, 44)
	frameIconsContainer.BackgroundTransparency = 1
	frameIconsContainer.Parent = frameContent

	local frameIconsLayout = Instance.new("UIListLayout")
	frameIconsLayout.FillDirection = Enum.FillDirection.Horizontal
	frameIconsLayout.SortOrder = Enum.SortOrder.LayoutOrder
	frameIconsLayout.Padding = UDim.new(0, 10)
	frameIconsLayout.Parent = frameIconsContainer
	
	local frameVerifiedBox = createIconBox("Show Verified (True/False)", 1, frameIconsContainer)
	local framePremiumBox = createIconBox("Show Premium (True/False)", 2, frameIconsContainer)
	
	local function updateFrameIcons()
		local targetText = targetBox.Text:gsub("^%s+", ""):gsub("%s+$", "")
		-- Note: We can't easily get the target player instance here without validation
		-- so we'll apply these attributes when the "Frame" button is clicked
		-- OR we can try to find them now
		
		-- Just in case, store them in temporary storage if possible, or just rely on the inputs being read during application
	end
	
	-- We will read these values in the Apply Button logic and set attributes then

	local sourceBox, sourceContainer = makeBox("Enter Source UserID ", 3, "â€¢")
	sourceContainer.Parent = frameContent

	local tgtLabel = makeLabel("Target Player", 4, "â€¢")
	tgtLabel.Parent = frameContent

	local targetBox, targetContainer = makeBox("Enter Target UserID or Username", 5, "â€¢")
	targetContainer.Parent = frameContent

	-- Wins and Elims container (side by side)
	local statsContainer = Instance.new("Frame")
	statsContainer.Name = "StatsContainer"
	statsContainer.LayoutOrder = 6
	statsContainer.Size = UDim2.new(1, 0, 0, 50)
	statsContainer.BackgroundTransparency = 1
	statsContainer.Parent = frameContent

	local statsLayout = Instance.new("UIListLayout")
	statsLayout.FillDirection = Enum.FillDirection.Horizontal
	statsLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	statsLayout.VerticalAlignment = Enum.VerticalAlignment.Center
	statsLayout.Padding = UDim.new(0, 10)
	statsLayout.Parent = statsContainer

	-- Wins field
	local winsFrame = Instance.new("Frame")
	winsFrame.Name = "WinsFrame"
	winsFrame.Size = UDim2.new(0.48, 0, 1, 0)
	winsFrame.BackgroundTransparency = 1
	winsFrame.Parent = statsContainer

	local winsLabel = Instance.new("TextLabel")
	winsLabel.Size = UDim2.new(1, 0, 0, 18)
	winsLabel.BackgroundTransparency = 1
	winsLabel.Text = "â€¢ Wins"
	winsLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	winsLabel.Font = Enum.Font.GothamSemibold
	winsLabel.TextSize = 13
	winsLabel.TextXAlignment = Enum.TextXAlignment.Left
	winsLabel.Parent = winsFrame

	local winsBox = Instance.new("TextBox")
	winsBox.Size = UDim2.new(1, 0, 0, 32)
	winsBox.Position = UDim2.new(0, 0, 0, 18)
	winsBox.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	winsBox.TextColor3 = Color3.fromRGB(240, 240, 245)
	winsBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
	winsBox.PlaceholderText = "â€¢ Enter Wins"
	winsBox.Font = Enum.Font.GothamMedium
	winsBox.TextSize = 13
	winsBox.ClearTextOnFocus = false
	winsBox.Text = ""
	winsBox.TextXAlignment = Enum.TextXAlignment.Left
	winsBox.Parent = winsFrame

	winsBox:GetPropertyChangedSignal("Text"):Connect(function()
		if #winsBox.Text > 20 then winsBox.Text = winsBox.Text:sub(1, 20) end
	end)

	local winsCorner = Instance.new("UICorner")
	winsCorner.CornerRadius = UDim.new(0, 8)
	winsCorner.Parent = winsBox

	local winsStroke = Instance.new("UIStroke")
	winsStroke.Color = Color3.fromRGB(65, 65, 85)
	winsStroke.Thickness = 1
	winsStroke.Transparency = 0.6
	winsStroke.Parent = winsBox

	local winsPadding = Instance.new("UIPadding")
	winsPadding.PaddingLeft = UDim.new(0, 10)
	winsPadding.Parent = winsBox

	-- Wins focus effects
	winsBox.Focused:Connect(function()
		game:GetService("TweenService"):Create(winsStroke, TweenInfo.new(0.2), {
			Transparency = 0.2,
			Color = Color3.fromRGB(80, 120, 200)
		}):Play()
	end)
	winsBox.FocusLost:Connect(function()
		game:GetService("TweenService"):Create(winsStroke, TweenInfo.new(0.2), {
			Transparency = 0.6,
			Color = Color3.fromRGB(65, 65, 85)
		}):Play()
	end)

	-- Elims field
	local elimsFrame = Instance.new("Frame")
	elimsFrame.Name = "ElimsFrame"
	elimsFrame.Size = UDim2.new(0.48, 0, 1, 0)
	elimsFrame.BackgroundTransparency = 1
	elimsFrame.Parent = statsContainer

	local elimsLabel = Instance.new("TextLabel")
	elimsLabel.Size = UDim2.new(1, 0, 0, 18)
	elimsLabel.BackgroundTransparency = 1
	elimsLabel.Text = "â€¢ Elims"
	elimsLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	elimsLabel.Font = Enum.Font.GothamSemibold
	elimsLabel.TextSize = 13
	elimsLabel.TextXAlignment = Enum.TextXAlignment.Left
	elimsLabel.Parent = elimsFrame

	local elimsBox = Instance.new("TextBox")
	elimsBox.Size = UDim2.new(1, 0, 0, 32)
	elimsBox.Position = UDim2.new(0, 0, 0, 18)
	elimsBox.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	elimsBox.TextColor3 = Color3.fromRGB(240, 240, 245)
	elimsBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
	elimsBox.PlaceholderText = "â€¢ Enter Elims"
	elimsBox.Font = Enum.Font.GothamMedium
	elimsBox.TextSize = 13
	elimsBox.ClearTextOnFocus = false
	elimsBox.Text = ""
	elimsBox.TextXAlignment = Enum.TextXAlignment.Left
	elimsBox.Parent = elimsFrame

	elimsBox:GetPropertyChangedSignal("Text"):Connect(function()
		if #elimsBox.Text > 20 then elimsBox.Text = elimsBox.Text:sub(1, 20) end
	end)

	local elimsCorner = Instance.new("UICorner")
	elimsCorner.CornerRadius = UDim.new(0, 8)
	elimsCorner.Parent = elimsBox

	local elimsStroke = Instance.new("UIStroke")
	elimsStroke.Color = Color3.fromRGB(65, 65, 85)
	elimsStroke.Thickness = 1
	elimsStroke.Transparency = 0.6
	elimsStroke.Parent = elimsBox

	local elimsPadding = Instance.new("UIPadding")
	elimsPadding.PaddingLeft = UDim.new(0, 10)
	elimsPadding.Parent = elimsBox

	-- Elims focus effects
	elimsBox.Focused:Connect(function()
		game:GetService("TweenService"):Create(elimsStroke, TweenInfo.new(0.2), {
			Transparency = 0.2,
			Color = Color3.fromRGB(80, 120, 200)
		}):Play()
	end)
	elimsBox.FocusLost:Connect(function()
		game:GetService("TweenService"):Create(elimsStroke, TweenInfo.new(0.2), {
			Transparency = 0.6,
			Color = Color3.fromRGB(65, 65, 85)
		}):Play()
	end)

	-- Apply button
	local applyButton = Instance.new("TextButton")
	applyButton.LayoutOrder = 7
	applyButton.Size = UDim2.new(1, 0, 0, 42)
	applyButton.BackgroundColor3 = Color3.fromRGB(25, 60, 180)
	applyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
	applyButton.Font = Enum.Font.GothamBold
	applyButton.TextSize = 15
	applyButton.Text = "Frame"
	applyButton.AutoButtonColor = false
	applyButton.Parent = frameContent

	local applyCorner = Instance.new("UICorner")
	applyCorner.CornerRadius = UDim.new(0, 12)
	applyCorner.Parent = applyButton

	local applyGradient = Instance.new("UIGradient")
	applyGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 70, 200)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(25, 60, 180)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(20, 50, 160))
	}
	applyGradient.Rotation = 90
	applyGradient.Parent = applyButton

	local applyStroke = Instance.new("UIStroke")
	applyStroke.Color = Color3.fromRGB(60, 90, 220)
	applyStroke.Thickness = 1
	applyStroke.Transparency = 0.7
	applyStroke.Parent = applyButton

	-- Apply button hover effects
	local originalApplyColor = applyGradient.Color
	local hoverApplyColor = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(45, 80, 220)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(35, 70, 200)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 60, 180))
	}

	local originalApplyStrokeColor = applyStroke.Color
	local hoverApplyStrokeColor = Color3.fromRGB(80, 110, 240)

	applyButton.MouseEnter:Connect(function()
		local tween = game:GetService("TweenService"):Create(applyGradient, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {Color = hoverApplyColor})
		tween:Play()
		local strokeTween = game:GetService("TweenService"):Create(applyStroke, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
			Transparency = 0.3,
			Color = hoverApplyStrokeColor
		})
		strokeTween:Play()
	end)

	applyButton.MouseLeave:Connect(function()
		local tween = game:GetService("TweenService"):Create(applyGradient, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {Color = originalApplyColor})
		tween:Play()
		local strokeTween = game:GetService("TweenService"):Create(applyStroke, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
			Transparency = 0.7,
			Color = originalApplyStrokeColor
		})
		strokeTween:Play()
	end)

	local function setStatus(text, color, duration, progress)
		status.Text = text
		status.TextColor3 = color

		-- Update progress bar
		if progress then
			local tween = game:GetService("TweenService"):Create(progressBar, TweenInfo.new(0.3), {
				Size = UDim2.new(progress, 0, 0, 2)
			})
			tween:Play()
		else
			local tween = game:GetService("TweenService"):Create(progressBar, TweenInfo.new(0.3), {
				Size = UDim2.new(0, 0, 0, 2)
			})
			tween:Play()
		end

		if duration then
			task.spawn(function()
				task.wait(duration)
				status.Text = "Ready"
				status.TextColor3 = Color3.fromRGB(180, 220, 180)
				local tween = game:GetService("TweenService"):Create(progressBar, TweenInfo.new(0.3), {
					Size = UDim2.new(0, 0, 0, 2)
				})
				tween:Play()
			end)
		end
	end

	local function createLoadingAnimation(button, originalText)
		local loadingDots = 0
		local loadingConnection
		loadingConnection = game:GetService("RunService").Heartbeat:Connect(function()
			loadingDots = (loadingDots + 1) % 4
			local dots = string.rep("â—", loadingDots) .. string.rep("â—‹", 3 - loadingDots)
			button.Text = "Processing" .. dots
		end)
		return loadingConnection
	end

	applyButton.MouseButton1Click:Connect(function()
		local sourceText = sourceBox.Text:gsub("%s+", "")
		local targetText = targetBox.Text:gsub("^%s+", ""):gsub("%s+$", "")

		if sourceText == "" then
			setStatus("Please enter a source player!", Color3.fromRGB(255, 120, 120), 3)
			return
		end

		-- Validate source UserID
		local sourceValid, sourceError, sourceUserId = validateUserId(sourceText)
		if not sourceValid then
			setStatus("Source " .. sourceError, Color3.fromRGB(255, 120, 120), 3)
			return
		end

		-- Validate target UserID or find by username
		local targetPlayer
		local targetUserId

		if tonumber(targetText) then
			-- Target is a UserID
			local targetValid, targetError, parsedUserId = validateUserId(targetText)
			if not targetValid then
				setStatus("Target " .. targetError, Color3.fromRGB(255, 120, 120), 3)
				return
			end
			targetUserId = parsedUserId
			targetPlayer = getTargetPlayer(tostring(targetUserId))
		else
			-- Target is a username
			targetPlayer = getTargetPlayer(targetText)
			if targetPlayer then
				targetUserId = targetPlayer.UserId
			end
		end

		-- Try to resolve source player
		local sourcePlayer
		local actualSourceUserId = sourceUserId

		if sourceUserId then
			sourcePlayer = getTargetPlayer(tostring(sourceUserId))
		else
			-- Username entered - try to find player in game
			sourcePlayer = getTargetPlayer(sourceText)
			if sourcePlayer then
				actualSourceUserId = sourcePlayer.UserId
			end
		end

		if not sourcePlayer and not actualSourceUserId then
			setStatus("Source player not found in game! Enter UserID or ensure player is in game.", Color3.fromRGB(255, 120, 120), 3)
			return
		end

		setStatus("Preparing avatar copy...", Color3.fromRGB(255, 200, 120))
		applyButton.Active = false

		local loadingAnim = createLoadingAnimation(applyButton, applyButton.Text)

		if not targetPlayer then
			loadingAnim:Disconnect()
			applyButton.Text = "Frame"
			applyButton.Active = true
			setStatus("Target player not found: " .. targetText .. " (must be in game)", Color3.fromRGB(255, 120, 120), 3)
			return
		end

		setStatus("Fetching avatar data...", Color3.fromRGB(255, 220, 140), nil, 0.2)

		local success, result, errorMsg = pcall(function()
			local userId = actualSourceUserId or (sourcePlayer and sourcePlayer.UserId)

			-- Update progress during copying
			setStatus("Copying avatar appearance...", Color3.fromRGB(255, 220, 140), nil, 0.6)
			return copyAvatarToPlayer(userId, targetPlayer)
		end)

		if success and result then
			setStatus("Setting up UI monitoring...", Color3.fromRGB(255, 220, 140), nil, 0.8)
		end

		loadingAnim:Disconnect()

		if success and result then
			setStatus("Avatar applied successfully to " .. targetPlayer.Name, Color3.fromRGB(25, 60, 180), 4, 1.0)
			applyButton.Text = "âœ“ Applied!"
			log(LOG_LEVEL.INFO, "Successfully applied avatar from UserId %d to %s", actualSourceUserId or sourceUserId, targetPlayer.Name)
			
			-- Apply Icon Attributes
			local verified = frameVerifiedBox.Text:lower() == "true"
			local premium = framePremiumBox.Text:lower() == "true"
			targetPlayer:SetAttribute("ShowVerified", verified)
			targetPlayer:SetAttribute("ShowPremium", premium)
			
			-- Also apply stat spoofing if values are entered
			local winsValue = parseStatInput(winsBox.Text)
			local elimsValue = parseStatInput(elimsBox.Text)
			
			if winsValue then
				local statSuccess, statErr = spoofLeaderstats(targetPlayer, "Wins", winsValue)
				if statSuccess then
					startStatMonitor(targetPlayer, "Wins", winsValue)
					log(LOG_LEVEL.INFO, "Spoofed Wins to %s for %s", formatStatNumber(winsValue), targetPlayer.Name)
				else
					log(LOG_LEVEL.WARN, "Failed to spoof Wins: %s", tostring(statErr))
				end
			end
			
			if elimsValue then
				local statSuccess, statErr = spoofLeaderstats(targetPlayer, "Elims", elimsValue)
				if statSuccess then
					startStatMonitor(targetPlayer, "Elims", elimsValue)
					log(LOG_LEVEL.INFO, "Spoofed Elims to %s for %s", formatStatNumber(elimsValue), targetPlayer.Name)
				else
					log(LOG_LEVEL.WARN, "Failed to spoof Elims: %s", tostring(statErr))
				end
			end
			
			task.wait(1.5)
			applyButton.Text = "Frame"
		else
			local errorText = "Failed to apply avatar"
			local detailedError = ""

			if type(result) == "string" then
				errorText = result
			elseif errorMsg and type(errorMsg) == "string" then
				errorText = errorMsg
			elseif not success then
				errorText = "Unknown error occurred"
				detailedError = tostring(result)
			end

			-- Provide helpful suggestions based on error type
			if errorText:find("character") then
				errorText = errorText .. " - Target player may have left the game"
			elseif errorText:find("avatar") then
				errorText = errorText .. " - Source avatar may be private or invalid"
			elseif errorText:find("respawn") then
				errorText = errorText .. " - Try again after target stops moving"
			end

			setStatus(errorText, Color3.fromRGB(255, 120, 120), 6, 0)
			applyButton.Text = "âœ— Failed"
			log(LOG_LEVEL.ERROR, "Avatar copy failed: %s", errorText)
			if detailedError ~= "" then
				log(LOG_LEVEL.DEBUG, "Detailed error: %s", detailedError)
			end

			task.wait(2)
			applyButton.Text = "Frame"
		end

		applyButton.Active = true
	end)

	-- Revert button
	local revertButton = Instance.new("TextButton")
	revertButton.LayoutOrder = 8
	revertButton.Size = UDim2.new(1, 0, 0, 38)
	revertButton.BackgroundColor3 = Color3.fromRGB(85, 30, 30)
	revertButton.TextColor3 = Color3.fromRGB(255, 255, 255)
	revertButton.Font = Enum.Font.GothamBold
	revertButton.TextSize = 14
	revertButton.Text = "Revert Changes"
	revertButton.AutoButtonColor = false
	revertButton.Parent = frameContent

	local revertCorner = Instance.new("UICorner")
	revertCorner.CornerRadius = UDim.new(0, 10)
	revertCorner.Parent = revertButton

	local revertGradient = Instance.new("UIGradient")
	revertGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 40, 40)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(85, 30, 30)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 25, 25))
	}
	revertGradient.Rotation = 90
	revertGradient.Parent = revertButton

	local revertStroke = Instance.new("UIStroke")
	revertStroke.Color = Color3.fromRGB(130, 50, 50)
	revertStroke.Thickness = 1
	revertStroke.Transparency = 0.7
	revertStroke.Parent = revertButton

	-- Revert button hover effects
	local originalRevertColor = revertGradient.Color
	local hoverRevertColor = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(120, 50, 50)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(100, 40, 40)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(85, 35, 35))
	}

	local originalRevertStrokeColor = revertStroke.Color
	local hoverRevertStrokeColor = Color3.fromRGB(150, 60, 60)

	revertButton.MouseEnter:Connect(function()
		local tween = game:GetService("TweenService"):Create(revertGradient, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {Color = hoverRevertColor})
		tween:Play()
		local strokeTween = game:GetService("TweenService"):Create(revertStroke, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
			Transparency = 0.3,
			Color = hoverRevertStrokeColor
		})
		strokeTween:Play()
	end)

	revertButton.MouseLeave:Connect(function()
		local tween = game:GetService("TweenService"):Create(revertGradient, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {Color = originalRevertColor})
		tween:Play()
		local strokeTween = game:GetService("TweenService"):Create(revertStroke, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
			Transparency = 0.7,
			Color = originalRevertStrokeColor
		})
		strokeTween:Play()
	end)

	revertButton.MouseButton1Click:Connect(function()
		local targetText = targetBox.Text:gsub("^%s+", ""):gsub("%s+$", "")

		local target = getTargetPlayer(targetText)
		if not target then
			setStatus("Target player not found: " .. targetText, Color3.fromRGB(255, 120, 120), 3)
			return
		end

		setStatus("Reverting changes for " .. target.Name .. "...", Color3.fromRGB(255, 200, 120))
		revertButton.Active = false

		local loadingAnim = createLoadingAnimation(revertButton, revertButton.Text)

		-- Stop stat monitors for this player
		stopAllStatMonitors(target)
		
		-- Revert stats
		local statsReverted = revertLeaderstats(target)
		
		-- Revert avatar
		local success, result = pcall(function()
			return revertPlayer(target)
		end)

		loadingAnim:Disconnect()

		if success and result then
			local msg = "Reverted for " .. target.Name
			if statsReverted then
				msg = msg .. " (avatar + stats)"
			end
			setStatus(msg, Color3.fromRGB(25, 60, 180), 4)
			revertButton.Text = "Reverted!"
			task.wait(1)
			revertButton.Text = "Revert Changes"
		else
			setStatus("Failed to revert. Check console.", Color3.fromRGB(255, 120, 120), 4)
			revertButton.Text = "Failed"
			task.wait(1)
			revertButton.Text = "Revert Changes"
		end

		revertButton.Active = true
	end)

	-- ============================================
	-- SELF TAB CONTENT
	-- ============================================

	-- Source field (large)
	local selfSrcLabel = Instance.new("TextLabel")
	selfSrcLabel.LayoutOrder = 1
	selfSrcLabel.Size = UDim2.new(1, 0, 0, 18)
	selfSrcLabel.BackgroundTransparency = 1
	selfSrcLabel.Text = "â€¢ Source Player"
	selfSrcLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	selfSrcLabel.Font = Enum.Font.GothamSemibold
	selfSrcLabel.TextSize = 13
	selfSrcLabel.TextXAlignment = Enum.TextXAlignment.Left
	selfSrcLabel.Parent = selfContent
	
	-- Icons for Self Tab
	local selfIconsContainer = Instance.new("Frame")
	selfIconsContainer.LayoutOrder = 0
	selfIconsContainer.Size = UDim2.new(1, 0, 0, 44)
	selfIconsContainer.BackgroundTransparency = 1
	selfIconsContainer.Parent = selfContent

	local selfIconsLayout = Instance.new("UIListLayout")
	selfIconsLayout.FillDirection = Enum.FillDirection.Horizontal
	selfIconsLayout.SortOrder = Enum.SortOrder.LayoutOrder
	selfIconsLayout.Padding = UDim.new(0, 10)
	selfIconsLayout.Parent = selfIconsContainer
	
	local selfVerifiedBox = createIconBox("Show Verified (True/False)", 1, selfIconsContainer)
	local selfPremiumBox = createIconBox("Show Premium (True/False)", 2, selfIconsContainer)
	
	local function updateSelfIcons()
		local verified = selfVerifiedBox.Text:lower() == "true"
		local premium = selfPremiumBox.Text:lower() == "true"
		LocalPlayer:SetAttribute("ShowVerified", verified)
		LocalPlayer:SetAttribute("ShowPremium", premium)
	end
	
	selfVerifiedBox:GetPropertyChangedSignal("Text"):Connect(updateSelfIcons)
	selfPremiumBox:GetPropertyChangedSignal("Text"):Connect(updateSelfIcons)

	local selfSrcLabelStroke = Instance.new("UIStroke")
	selfSrcLabelStroke.Color = Color3.fromRGB(0, 0, 0)
	selfSrcLabelStroke.Thickness = 0.5
	selfSrcLabelStroke.Transparency = 0.7
	selfSrcLabelStroke.Parent = selfSrcLabel

	local selfSourceContainer = Instance.new("Frame")
	selfSourceContainer.LayoutOrder = 2
	selfSourceContainer.Size = UDim2.new(1, 0, 0, 44)
	selfSourceContainer.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	selfSourceContainer.BorderSizePixel = 0
	selfSourceContainer.Parent = selfContent

	local selfSrcCorner = Instance.new("UICorner")
	selfSrcCorner.CornerRadius = UDim.new(0, 10)
	selfSrcCorner.Parent = selfSourceContainer

	local selfSrcStroke = Instance.new("UIStroke")
	selfSrcStroke.Color = Color3.fromRGB(65, 65, 85)
	selfSrcStroke.Thickness = 1
	selfSrcStroke.Transparency = 0.6
	selfSrcStroke.Parent = selfSourceContainer

	local selfSourceBox = Instance.new("TextBox")
	selfSourceBox.Size = UDim2.new(1, 0, 1, 0)
	selfSourceBox.BackgroundTransparency = 1
	selfSourceBox.TextColor3 = Color3.fromRGB(240, 240, 245)
	selfSourceBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
	selfSourceBox.PlaceholderText = "â€¢ Enter UserID"
	selfSourceBox.Font = Enum.Font.GothamMedium
	selfSourceBox.TextSize = 14
	selfSourceBox.ClearTextOnFocus = false
	selfSourceBox.Text = ""
	selfSourceBox.TextXAlignment = Enum.TextXAlignment.Left
	selfSourceBox.Parent = selfSourceContainer

	selfSourceBox:GetPropertyChangedSignal("Text"):Connect(function()
		if #selfSourceBox.Text > 20 then selfSourceBox.Text = selfSourceBox.Text:sub(1, 20) end
	end)

	local selfSrcPad = Instance.new("UIPadding")
	selfSrcPad.PaddingLeft = UDim.new(0, 14)
	selfSrcPad.PaddingRight = UDim.new(0, 14)
	selfSrcPad.Parent = selfSourceBox

	selfSourceBox.Focused:Connect(function()
		game:GetService("TweenService"):Create(selfSrcStroke, TweenInfo.new(0.2), {
			Transparency = 0,
			Color = Color3.fromRGB(60, 100, 220),
			Thickness = 2
		}):Play()
	end)
	selfSourceBox.FocusLost:Connect(function()
		game:GetService("TweenService"):Create(selfSrcStroke, TweenInfo.new(0.2), {
			Transparency = 0.6,
			Color = Color3.fromRGB(65, 65, 85),
			Thickness = 1
		}):Play()
	end)

	-- Stats row: Wins + Coins on left, Elims on right
	local selfStatsContainer = Instance.new("Frame")
	selfStatsContainer.Name = "SelfStatsContainer"
	selfStatsContainer.LayoutOrder = 3
	selfStatsContainer.Size = UDim2.new(1, 0, 0, 145)
	selfStatsContainer.BackgroundTransparency = 1
	selfStatsContainer.Parent = selfContent

	local selfStatsLayout = Instance.new("UIListLayout")
	selfStatsLayout.FillDirection = Enum.FillDirection.Horizontal
	selfStatsLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	selfStatsLayout.Padding = UDim.new(0, 10)
	selfStatsLayout.Parent = selfStatsContainer

	-- Left column (Wins + Coins stacked)
	local leftColumn = Instance.new("Frame")
	leftColumn.Name = "LeftColumn"
	leftColumn.Size = UDim2.new(0.48, 0, 1, 0)
	leftColumn.BackgroundTransparency = 1
	leftColumn.Parent = selfStatsContainer

	local leftLayout = Instance.new("UIListLayout")
	leftLayout.FillDirection = Enum.FillDirection.Vertical
	leftLayout.SortOrder = Enum.SortOrder.LayoutOrder
	leftLayout.Padding = UDim.new(0, 6)
	leftLayout.Parent = leftColumn

	-- Self Wins field
	local selfWinsLabel = Instance.new("TextLabel")
	selfWinsLabel.LayoutOrder = 1
	selfWinsLabel.Size = UDim2.new(1, 0, 0, 16)
	selfWinsLabel.BackgroundTransparency = 1
	selfWinsLabel.Text = "â€¢ Wins"
	selfWinsLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	selfWinsLabel.Font = Enum.Font.GothamSemibold
	selfWinsLabel.TextSize = 13
	selfWinsLabel.TextXAlignment = Enum.TextXAlignment.Left
	selfWinsLabel.Parent = leftColumn

	local selfWinsBox = Instance.new("TextBox")
	selfWinsBox.LayoutOrder = 2
	selfWinsBox.Size = UDim2.new(1, 0, 0, 32)
	selfWinsBox.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	selfWinsBox.TextColor3 = Color3.fromRGB(240, 240, 245)
	selfWinsBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
	selfWinsBox.PlaceholderText = "â€¢ Enter Wins"
	selfWinsBox.Font = Enum.Font.GothamMedium
	selfWinsBox.TextSize = 13
	selfWinsBox.ClearTextOnFocus = false
	selfWinsBox.Text = ""
	selfWinsBox.TextXAlignment = Enum.TextXAlignment.Left
	selfWinsBox.Parent = leftColumn

	selfWinsBox:GetPropertyChangedSignal("Text"):Connect(function()
		if #selfWinsBox.Text > 20 then selfWinsBox.Text = selfWinsBox.Text:sub(1, 20) end
	end)

	local selfWinsCorner = Instance.new("UICorner")
	selfWinsCorner.CornerRadius = UDim.new(0, 8)
	selfWinsCorner.Parent = selfWinsBox

	local selfWinsStroke = Instance.new("UIStroke")
	selfWinsStroke.Color = Color3.fromRGB(65, 65, 85)
	selfWinsStroke.Thickness = 1
	selfWinsStroke.Transparency = 0.6
	selfWinsStroke.Parent = selfWinsBox

	local selfWinsPadding = Instance.new("UIPadding")
	selfWinsPadding.PaddingLeft = UDim.new(0, 10)
	selfWinsPadding.Parent = selfWinsBox

	selfWinsBox.Focused:Connect(function()
		game:GetService("TweenService"):Create(selfWinsStroke, TweenInfo.new(0.2), {
			Transparency = 0.2,
			Color = Color3.fromRGB(80, 120, 200)
		}):Play()
	end)
	selfWinsBox.FocusLost:Connect(function()
		game:GetService("TweenService"):Create(selfWinsStroke, TweenInfo.new(0.2), {
			Transparency = 0.6,
			Color = Color3.fromRGB(65, 65, 85)
		}):Play()
	end)

	-- Coins field (under wins)
	local selfCoinsLabel = Instance.new("TextLabel")
	selfCoinsLabel.LayoutOrder = 3
	selfCoinsLabel.Size = UDim2.new(1, 0, 0, 16)
	selfCoinsLabel.BackgroundTransparency = 1
	selfCoinsLabel.Text = "â€¢ Coins"
	selfCoinsLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	selfCoinsLabel.Font = Enum.Font.GothamSemibold
	selfCoinsLabel.TextSize = 13
	selfCoinsLabel.TextXAlignment = Enum.TextXAlignment.Left
	selfCoinsLabel.Parent = leftColumn

	local selfCoinsBox = Instance.new("TextBox")
	selfCoinsBox.LayoutOrder = 4
	selfCoinsBox.Size = UDim2.new(1, 0, 0, 32)
	selfCoinsBox.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	selfCoinsBox.TextColor3 = Color3.fromRGB(240, 240, 245)
	selfCoinsBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
	selfCoinsBox.PlaceholderText = "â€¢ Enter Coins"
	selfCoinsBox.Font = Enum.Font.GothamMedium
	selfCoinsBox.TextSize = 13
	selfCoinsBox.ClearTextOnFocus = false
	selfCoinsBox.Text = ""
	selfCoinsBox.TextXAlignment = Enum.TextXAlignment.Left
	selfCoinsBox.Parent = leftColumn

	selfCoinsBox:GetPropertyChangedSignal("Text"):Connect(function()
		if #selfCoinsBox.Text > 20 then selfCoinsBox.Text = selfCoinsBox.Text:sub(1, 20) end
	end)

	local selfCoinsCorner = Instance.new("UICorner")
	selfCoinsCorner.CornerRadius = UDim.new(0, 8)
	selfCoinsCorner.Parent = selfCoinsBox

	local selfCoinsStroke = Instance.new("UIStroke")
	selfCoinsStroke.Color = Color3.fromRGB(65, 65, 85)
	selfCoinsStroke.Thickness = 1
	selfCoinsStroke.Transparency = 0.6
	selfCoinsStroke.Parent = selfCoinsBox

	local selfCoinsPadding = Instance.new("UIPadding")
	selfCoinsPadding.PaddingLeft = UDim.new(0, 10)
	selfCoinsPadding.Parent = selfCoinsBox

	selfCoinsBox.Focused:Connect(function()
		game:GetService("TweenService"):Create(selfCoinsStroke, TweenInfo.new(0.2), {
			Transparency = 0.2,
			Color = Color3.fromRGB(80, 120, 200)
		}):Play()
	end)
	selfCoinsBox.FocusLost:Connect(function()
		game:GetService("TweenService"):Create(selfCoinsStroke, TweenInfo.new(0.2), {
			Transparency = 0.6,
			Color = Color3.fromRGB(65, 65, 85)
		}):Play()
	end)

	-- Right column (Elims)
	local rightColumn = Instance.new("Frame")
	rightColumn.Name = "RightColumn"
	rightColumn.Size = UDim2.new(0.48, 0, 1, 0)
	rightColumn.BackgroundTransparency = 1
	rightColumn.Parent = selfStatsContainer

	local rightLayout = Instance.new("UIListLayout")
	rightLayout.FillDirection = Enum.FillDirection.Vertical
	rightLayout.SortOrder = Enum.SortOrder.LayoutOrder
	rightLayout.Padding = UDim.new(0, 6)
	rightLayout.Parent = rightColumn

	-- Self Elims field
	local selfElimsLabel = Instance.new("TextLabel")
	selfElimsLabel.LayoutOrder = 1
	selfElimsLabel.Size = UDim2.new(1, 0, 0, 16)
	selfElimsLabel.BackgroundTransparency = 1
	selfElimsLabel.Text = "â€¢ Elims"
	selfElimsLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	selfElimsLabel.Font = Enum.Font.GothamSemibold
	selfElimsLabel.TextSize = 13
	selfElimsLabel.TextXAlignment = Enum.TextXAlignment.Left
	selfElimsLabel.Parent = rightColumn

	local selfElimsBox = Instance.new("TextBox")
	selfElimsBox.LayoutOrder = 2
	selfElimsBox.Size = UDim2.new(1, 0, 0, 32)
	selfElimsBox.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	selfElimsBox.TextColor3 = Color3.fromRGB(240, 240, 245)
	selfElimsBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
	selfElimsBox.PlaceholderText = "â€¢ Enter Elims"
	selfElimsBox.Font = Enum.Font.GothamMedium
	selfElimsBox.TextSize = 13
	selfElimsBox.ClearTextOnFocus = false
	selfElimsBox.Text = ""
	selfElimsBox.TextXAlignment = Enum.TextXAlignment.Left
	selfElimsBox.Parent = rightColumn

	selfElimsBox:GetPropertyChangedSignal("Text"):Connect(function()
		if #selfElimsBox.Text > 20 then selfElimsBox.Text = selfElimsBox.Text:sub(1, 20) end
	end)

	local selfElimsCorner = Instance.new("UICorner")
	selfElimsCorner.CornerRadius = UDim.new(0, 8)
	selfElimsCorner.Parent = selfElimsBox

	local selfElimsStroke = Instance.new("UIStroke")
	selfElimsStroke.Color = Color3.fromRGB(65, 65, 85)
	selfElimsStroke.Thickness = 1
	selfElimsStroke.Transparency = 0.6
	selfElimsStroke.Parent = selfElimsBox

	local selfElimsPadding = Instance.new("UIPadding")
	selfElimsPadding.PaddingLeft = UDim.new(0, 10)
	selfElimsPadding.Parent = selfElimsBox

	selfElimsBox.Focused:Connect(function()
		game:GetService("TweenService"):Create(selfElimsStroke, TweenInfo.new(0.2), {
			Transparency = 0.2,
			Color = Color3.fromRGB(80, 120, 200)
		}):Play()
	end)
	selfElimsBox.FocusLost:Connect(function()
		game:GetService("TweenService"):Create(selfElimsStroke, TweenInfo.new(0.2), {
			Transparency = 0.6,
			Color = Color3.fromRGB(65, 65, 85)
		}):Play()
	end)

	-- Self Tab Spoof Button (same size as frame tab Apply button)
	local selfSpoofButton = Instance.new("TextButton")
	selfSpoofButton.LayoutOrder = 4
	selfSpoofButton.Size = UDim2.new(1, 0, 0, 48)
	selfSpoofButton.BackgroundColor3 = themeColor
	selfSpoofButton.TextColor3 = Color3.fromRGB(255, 255, 255)
	selfSpoofButton.Font = Enum.Font.GothamBold
	selfSpoofButton.TextSize = 16
	selfSpoofButton.Text = "Spoof"
	selfSpoofButton.AutoButtonColor = false
	selfSpoofButton.Parent = selfContent

	local selfSpoofCorner = Instance.new("UICorner")
	selfSpoofCorner.CornerRadius = UDim.new(0, 12)
	selfSpoofCorner.Parent = selfSpoofButton

	local selfSpoofGradient = Instance.new("UIGradient")
	selfSpoofGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 70, 200)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(25, 60, 180)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(20, 50, 160))
	}
	selfSpoofGradient.Rotation = 90
	selfSpoofGradient.Parent = selfSpoofButton

	local selfSpoofStroke = Instance.new("UIStroke")
	selfSpoofStroke.Color = Color3.fromRGB(60, 90, 220)
	selfSpoofStroke.Thickness = 1
	selfSpoofStroke.Transparency = 0.7
	selfSpoofStroke.Parent = selfSpoofButton

	selfSpoofButton.MouseEnter:Connect(function()
		game:GetService("TweenService"):Create(selfSpoofGradient, TweenInfo.new(0.25), {
			Color = ColorSequence.new{
				ColorSequenceKeypoint.new(0, Color3.fromRGB(45, 80, 220)),
				ColorSequenceKeypoint.new(0.5, Color3.fromRGB(35, 70, 200)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 60, 180))
			}
		}):Play()
	end)
	selfSpoofButton.MouseLeave:Connect(function()
		game:GetService("TweenService"):Create(selfSpoofGradient, TweenInfo.new(0.25), {
			Color = ColorSequence.new{
				ColorSequenceKeypoint.new(0, Color3.fromRGB(35, 70, 200)),
				ColorSequenceKeypoint.new(0.5, Color3.fromRGB(25, 60, 180)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(20, 50, 160))
			}
		}):Play()
	end)

	selfSpoofButton.MouseButton1Click:Connect(function()
		local sourceText = selfSourceBox.Text:gsub("%s+", "")
		local winsValue = parseStatInput(selfWinsBox.Text)
		local elimsValue = parseStatInput(selfElimsBox.Text)
		local coinsValue = parseStatInput(selfCoinsBox.Text)
		
		local hasSource = sourceText ~= ""
		local hasStats = winsValue or elimsValue or coinsValue
		
		if not hasSource and not hasStats then
			setStatus("Enter a UserID or stat values to spoof!", Color3.fromRGB(255, 120, 120), 3)
			return
		end
		
		selfSpoofButton.Active = false
		local originalText = selfSpoofButton.Text
		selfSpoofButton.Text = "Spoofing..."
		
		local spoofedItems = {}
		
		-- Spoof avatar if source UserID is provided
		if hasSource then
			local sourceValid, sourceError, sourceUserId = validateUserId(sourceText)
			if not sourceValid then
				setStatus("Source " .. sourceError, Color3.fromRGB(255, 120, 120), 3)
				selfSpoofButton.Text = originalText
				selfSpoofButton.Active = true
				return
			end
			
			setStatus("Spoofing your avatar...", Color3.fromRGB(255, 220, 140))
			
			local success, result = pcall(function()
				return copyAvatarToPlayer(sourceUserId, LocalPlayer)
			end)
			
			if success and result then
				table.insert(spoofedItems, "Avatar")
				log(LOG_LEVEL.INFO, "Self-spoofed avatar to UserId %d", sourceUserId)
			else
				setStatus("Failed to spoof avatar: " .. tostring(result), Color3.fromRGB(255, 120, 120), 4)
				selfSpoofButton.Text = originalText
				selfSpoofButton.Active = true
				return
			end
		end
		
		-- Spoof stats
		if winsValue then
			local success, err = spoofLeaderstats(LocalPlayer, "Wins", winsValue)
			if success then
				startStatMonitor(LocalPlayer, "Wins", winsValue)
				table.insert(spoofedItems, "Wins")
			else
				log(LOG_LEVEL.WARN, "Failed to spoof Wins: %s", tostring(err))
			end
		end
		
		if elimsValue then
			local success, err = spoofLeaderstats(LocalPlayer, "Elims", elimsValue)
			if success then
				startStatMonitor(LocalPlayer, "Elims", elimsValue)
				table.insert(spoofedItems, "Elims")
			else
				log(LOG_LEVEL.WARN, "Failed to spoof Elims: %s", tostring(err))
			end
		end
		
		if coinsValue then
			local success, err = spoofLeaderstats(LocalPlayer, "Coins", coinsValue)
			if success then
				startStatMonitor(LocalPlayer, "Coins", coinsValue)
				table.insert(spoofedItems, "Coins")
			else
				log(LOG_LEVEL.WARN, "Failed to spoof Coins: %s", tostring(err))
			end
		end
		
		if #spoofedItems > 0 then
			setStatus("Spoofed: " .. table.concat(spoofedItems, ", "), Color3.fromRGB(25, 60, 180), 4)
			selfSpoofButton.Text = "âœ“ Spoofed!"
			task.wait(1.5)
		else
			setStatus("Nothing to spoof!", Color3.fromRGB(255, 200, 120), 3)
		end
		
		selfSpoofButton.Text = originalText
		selfSpoofButton.Active = true
	end)

	-- Self Tab Revert Button (same size as frame tab Revert button)
	local selfRevertButton = Instance.new("TextButton")
	selfRevertButton.LayoutOrder = 5
	selfRevertButton.Size = UDim2.new(1, 0, 0, 42)
	selfRevertButton.BackgroundColor3 = Color3.fromRGB(85, 30, 30)
	selfRevertButton.TextColor3 = Color3.fromRGB(255, 255, 255)
	selfRevertButton.Font = Enum.Font.GothamBold
	selfRevertButton.TextSize = 14
	selfRevertButton.Text = "Revert Changes"
	selfRevertButton.AutoButtonColor = false
	selfRevertButton.Parent = selfContent

	local selfRevertCorner = Instance.new("UICorner")
	selfRevertCorner.CornerRadius = UDim.new(0, 10)
	selfRevertCorner.Parent = selfRevertButton

	local selfRevertGradient = Instance.new("UIGradient")
	selfRevertGradient.Color = ColorSequence.new{
		ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 40, 40)),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(85, 30, 30)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 25, 25))
	}
	selfRevertGradient.Rotation = 90
	selfRevertGradient.Parent = selfRevertButton

	local selfRevertStroke = Instance.new("UIStroke")
	selfRevertStroke.Color = Color3.fromRGB(130, 50, 50)
	selfRevertStroke.Thickness = 1
	selfRevertStroke.Transparency = 0.7
	selfRevertStroke.Parent = selfRevertButton

	selfRevertButton.MouseEnter:Connect(function()
		game:GetService("TweenService"):Create(selfRevertGradient, TweenInfo.new(0.25), {
			Color = ColorSequence.new{
				ColorSequenceKeypoint.new(0, Color3.fromRGB(120, 50, 50)),
				ColorSequenceKeypoint.new(0.5, Color3.fromRGB(100, 40, 40)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(85, 35, 35))
			}
		}):Play()
	end)
	selfRevertButton.MouseLeave:Connect(function()
		game:GetService("TweenService"):Create(selfRevertGradient, TweenInfo.new(0.25), {
			Color = ColorSequence.new{
				ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 40, 40)),
				ColorSequenceKeypoint.new(0.5, Color3.fromRGB(85, 30, 30)),
				ColorSequenceKeypoint.new(1, Color3.fromRGB(70, 25, 25))
			}
		}):Play()
	end)

	selfRevertButton.MouseButton1Click:Connect(function()
		selfRevertButton.Active = false
		local originalText = selfRevertButton.Text
		selfRevertButton.Text = "Reverting..."
		
		local revertedItems = {}
		
		-- Stop all stat monitors for local player
		stopAllStatMonitors(LocalPlayer)
		
		-- Revert stats
		local success, result = revertLeaderstats(LocalPlayer)
		if success then
			table.insert(revertedItems, "Stats")
		end
		
		-- Revert avatar
		local revertSuccess = pcall(function()
			return revertPlayer(LocalPlayer)
		end)
		if revertSuccess then
			table.insert(revertedItems, "Avatar")
		end
		
		if #revertedItems > 0 then
			setStatus("Reverted: " .. table.concat(revertedItems, ", "), Color3.fromRGB(25, 60, 180), 4)
			selfRevertButton.Text = "âœ“ Reverted!"
			task.wait(1.5)
		else
			setStatus("Nothing to revert!", Color3.fromRGB(255, 200, 120), 3)
		end
		
		selfRevertButton.Text = originalText
		selfRevertButton.Active = true
	end)

	-- ============================================
	-- MISC TAB CONTENT
	-- ============================================

	-- Toggle Keybind Section
	local keybindLabel = Instance.new("TextLabel")
	keybindLabel.LayoutOrder = 1
	keybindLabel.Size = UDim2.new(1, 0, 0, 18)
	keybindLabel.BackgroundTransparency = 1
	keybindLabel.Text = "â€¢ Toggle Keybind"
	keybindLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	keybindLabel.Font = Enum.Font.GothamSemibold
	keybindLabel.TextSize = 13
	keybindLabel.TextXAlignment = Enum.TextXAlignment.Left
	keybindLabel.Parent = miscContent

	local keybindContainer = Instance.new("Frame")
	keybindContainer.LayoutOrder = 2
	keybindContainer.Size = UDim2.new(1, 0, 0, 42)
	keybindContainer.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	keybindContainer.BorderSizePixel = 0
	keybindContainer.Parent = miscContent

	local keybindCorner = Instance.new("UICorner")
	keybindCorner.CornerRadius = UDim.new(0, 10)
	keybindCorner.Parent = keybindContainer

	local keybindStroke = Instance.new("UIStroke")
	keybindStroke.Color = Color3.fromRGB(65, 65, 85)
	keybindStroke.Thickness = 1
	keybindStroke.Transparency = 0.6
	keybindStroke.Parent = keybindContainer

	local keybindText = Instance.new("TextLabel")
	keybindText.Size = UDim2.new(1, -20, 1, 0)
	keybindText.Position = UDim2.new(0, 10, 0, 0)
	keybindText.BackgroundTransparency = 1
	keybindText.TextColor3 = Color3.fromRGB(240, 240, 245)
	keybindText.Font = Enum.Font.GothamMedium
	keybindText.TextSize = 14
	keybindText.Text = "Ctrl + L"
	keybindText.TextXAlignment = Enum.TextXAlignment.Left
	keybindText.Parent = keybindContainer

	local keybindHint = Instance.new("TextLabel")
	keybindHint.Size = UDim2.new(0, 80, 1, 0)
	keybindHint.Position = UDim2.new(1, -90, 0, 0)
	keybindHint.BackgroundTransparency = 1
	keybindHint.TextColor3 = Color3.fromRGB(140, 140, 160)
	keybindHint.Font = Enum.Font.GothamMedium
	keybindHint.TextSize = 11
	keybindHint.Text = "Click to set"
	keybindHint.TextXAlignment = Enum.TextXAlignment.Right
	keybindHint.Parent = keybindContainer

	-- Keybind state
	local currentKeybind = {key = Enum.KeyCode.L, ctrl = true, shift = false, alt = false}
	local waitingForKeybind = false

	local function updateKeybindDisplay()
		local parts = {}
		if currentKeybind.ctrl then table.insert(parts, "Ctrl") end
		if currentKeybind.shift then table.insert(parts, "Shift") end
		if currentKeybind.alt then table.insert(parts, "Alt") end
		table.insert(parts, currentKeybind.key.Name)
		keybindText.Text = table.concat(parts, " + ")
	end

	local keybindButton = Instance.new("TextButton")
	keybindButton.Size = UDim2.new(1, 0, 1, 0)
	keybindButton.BackgroundTransparency = 1
	keybindButton.Text = ""
	keybindButton.Parent = keybindContainer

	keybindButton.MouseButton1Click:Connect(function()
		if waitingForKeybind then return end
		waitingForKeybind = true
		keybindText.Text = "Press any key..."
		keybindHint.Text = ""
		keybindStroke.Color = Color3.fromRGB(80, 120, 200)
		keybindStroke.Transparency = 0.2
	end)

	game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessed)
		if waitingForKeybind and input.UserInputType == Enum.UserInputType.Keyboard then
			-- Ignore modifier keys alone
			if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl or
			   input.KeyCode == Enum.KeyCode.LeftShift or input.KeyCode == Enum.KeyCode.RightShift or
			   input.KeyCode == Enum.KeyCode.LeftAlt or input.KeyCode == Enum.KeyCode.RightAlt then
				return
			end

			local uis = game:GetService("UserInputService")
			currentKeybind.key = input.KeyCode
			currentKeybind.ctrl = uis:IsKeyDown(Enum.KeyCode.LeftControl) or uis:IsKeyDown(Enum.KeyCode.RightControl)
			currentKeybind.shift = uis:IsKeyDown(Enum.KeyCode.LeftShift) or uis:IsKeyDown(Enum.KeyCode.RightShift)
			currentKeybind.alt = uis:IsKeyDown(Enum.KeyCode.LeftAlt) or uis:IsKeyDown(Enum.KeyCode.RightAlt)

			waitingForKeybind = false
			updateKeybindDisplay()
			keybindHint.Text = "Click to set"
			keybindStroke.Color = Color3.fromRGB(65, 65, 85)
			keybindStroke.Transparency = 0.6
		end
	end)

	-- Color Picker Section
	local colorLabel = Instance.new("TextLabel")
	colorLabel.LayoutOrder = 3
	colorLabel.Size = UDim2.new(1, 0, 0, 18)
	colorLabel.BackgroundTransparency = 1
	colorLabel.Text = "â€¢ Theme Color"
	colorLabel.TextColor3 = Color3.fromRGB(200, 200, 210)
	colorLabel.Font = Enum.Font.GothamSemibold
	colorLabel.TextSize = 13
	colorLabel.TextXAlignment = Enum.TextXAlignment.Left
	colorLabel.Parent = miscContent

	-- Color preview and hex input
	local colorRow = Instance.new("Frame")
	colorRow.LayoutOrder = 4
	colorRow.Size = UDim2.new(1, 0, 0, 40)
	colorRow.BackgroundTransparency = 1
	colorRow.Parent = miscContent

	local colorPreview = Instance.new("Frame")
	colorPreview.Size = UDim2.new(0, 40, 0, 40)
	colorPreview.Position = UDim2.new(0, 0, 0, 0)
	colorPreview.BackgroundColor3 = themeColor
	colorPreview.Parent = colorRow

	local colorPreviewCorner = Instance.new("UICorner")
	colorPreviewCorner.CornerRadius = UDim.new(0, 8)
	colorPreviewCorner.Parent = colorPreview

	local colorPreviewStroke = Instance.new("UIStroke")
	colorPreviewStroke.Color = Color3.fromRGB(100, 100, 120)
	colorPreviewStroke.Thickness = 2
	colorPreviewStroke.Parent = colorPreview

	local hexContainer = Instance.new("Frame")
	hexContainer.Size = UDim2.new(1, -54, 0, 40)
	hexContainer.Position = UDim2.new(0, 50, 0, 0)
	hexContainer.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
	hexContainer.Parent = colorRow

	local hexCorner = Instance.new("UICorner")
	hexCorner.CornerRadius = UDim.new(0, 8)
	hexCorner.Parent = hexContainer

	local hexStroke = Instance.new("UIStroke")
	hexStroke.Color = Color3.fromRGB(65, 65, 85)
	hexStroke.Thickness = 1
	hexStroke.Transparency = 0.6
	hexStroke.Parent = hexContainer

	local hexBox = Instance.new("TextBox")
	hexBox.Size = UDim2.new(1, 0, 1, 0)
	hexBox.BackgroundTransparency = 1
	hexBox.TextColor3 = Color3.fromRGB(240, 240, 245)
	hexBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 160)
	hexBox.PlaceholderText = "#193CB4"
	hexBox.Font = Enum.Font.GothamMedium
	hexBox.TextSize = 14
	hexBox.ClearTextOnFocus = false
	hexBox.Text = "#193CB4"
	hexBox.Parent = hexContainer

	local hexPad = Instance.new("UIPadding")
	hexPad.PaddingLeft = UDim.new(0, 12)
	hexPad.PaddingRight = UDim.new(0, 12)
	hexPad.Parent = hexBox

	hexBox.Focused:Connect(function()
		game:GetService("TweenService"):Create(hexStroke, TweenInfo.new(0.2), {
			Transparency = 0.2,
			Color = Color3.fromRGB(80, 120, 200)
		}):Play()
	end)
	hexBox.FocusLost:Connect(function()
		game:GetService("TweenService"):Create(hexStroke, TweenInfo.new(0.2), {
			Transparency = 0.6,
			Color = Color3.fromRGB(65, 65, 85)
		}):Play()
	end)

	-- Theme color is controlled via hex input only (sliders removed to reduce local variables)

	-- Function to update theme color across UI
	local function updateThemeColor(newColor)
		themeColor = newColor

		-- Update color preview
		colorPreview.BackgroundColor3 = newColor

		-- Update tab buttons
		for name, btn in pairs(tabButtons) do
			if name == currentTab then
				btn.BackgroundColor3 = newColor
			end
		end

		-- Update scrollbar colors
		for _, content in pairs(tabContents) do
			content.ScrollBarImageColor3 = newColor
		end

		-- Update title and subtitle colors
		if title then title.TextColor3 = newColor end
		if subtitle then subtitle.TextColor3 = newColor end

		-- Update apply button
		if applyButton then
			applyButton.BackgroundColor3 = newColor
			local gradient = applyButton:FindFirstChildOfClass("UIGradient")
			if gradient then
				gradient.Color = ColorSequence.new{
					ColorSequenceKeypoint.new(0, Color3.fromRGB(
						math.min(255, newColor.R * 255 + 10),
						math.min(255, newColor.G * 255 + 10),
						math.min(255, newColor.B * 255 + 20)
					)),
					ColorSequenceKeypoint.new(0.5, newColor),
					ColorSequenceKeypoint.new(1, Color3.fromRGB(
						math.max(0, newColor.R * 255 - 5),
						math.max(0, newColor.G * 255 - 10),
						math.max(0, newColor.B * 255 - 20)
					))
				}
			end
		end

		-- Update self spoof button
		if selfSpoofButton then
			selfSpoofButton.BackgroundColor3 = newColor
			local gradient = selfSpoofButton:FindFirstChildOfClass("UIGradient")
			if gradient then
				gradient.Color = ColorSequence.new{
					ColorSequenceKeypoint.new(0, Color3.fromRGB(
						math.min(255, newColor.R * 255 + 10),
						math.min(255, newColor.G * 255 + 10),
						math.min(255, newColor.B * 255 + 20)
					)),
					ColorSequenceKeypoint.new(0.5, newColor),
					ColorSequenceKeypoint.new(1, Color3.fromRGB(
						math.max(0, newColor.R * 255 - 5),
						math.max(0, newColor.G * 255 - 10),
						math.max(0, newColor.B * 255 - 20)
					))
				}
			end
		end

		-- Update minimize button to match theme
		if minimizeBtn then
			minimizeBtn.BackgroundColor3 = newColor
			local gradient = minimizeBtn:FindFirstChildOfClass("UIGradient")
			if gradient then
				gradient.Color = ColorSequence.new{
					ColorSequenceKeypoint.new(0, Color3.fromRGB(
						math.min(255, newColor.R * 255 + 35),
						math.min(255, newColor.G * 255 + 40),
						math.min(255, newColor.B * 255 + 50)
					)),
					ColorSequenceKeypoint.new(0.5, Color3.fromRGB(
						math.min(255, newColor.R * 255 + 10),
						math.min(255, newColor.G * 255 + 15),
						math.min(255, newColor.B * 255 + 20)
					)),
					ColorSequenceKeypoint.new(1, newColor)
				}
			end
		end

		-- Update selected tab button gradient
		if tabGradients and tabGradients[currentTab] then
			tabButtons[currentTab].BackgroundColor3 = newColor
			tabGradients[currentTab].Color = getSelectedGradient(newColor)
		end

		-- Update toggle buttons based on their state (theme when ON, red when OFF)
		local onGradient = ColorSequence.new{
			ColorSequenceKeypoint.new(0, Color3.fromRGB(
				math.min(255, newColor.R * 255 + 45),
				math.min(255, newColor.G * 255 + 60),
				math.min(255, newColor.B * 255 + 50)
			)),
			ColorSequenceKeypoint.new(0.5, Color3.fromRGB(
				math.min(255, newColor.R * 255 + 20),
				math.min(255, newColor.G * 255 + 30),
				math.min(255, newColor.B * 255 + 30)
			)),
			ColorSequenceKeypoint.new(1, Color3.fromRGB(
				math.min(255, newColor.R * 255 + 5),
				math.min(255, newColor.G * 255 + 10),
				math.min(255, newColor.B * 255 + 10)
			))
		}

		-- Respawn toggle (starts ON)
		if respawnGradient and autoRespawnEnabled then
			respawnGradient.Color = onGradient
			respawnToggle.BackgroundColor3 = newColor
		end

		-- Safe mode and delay only update if they're ON
		if safeGradient and safeModeEnabled then
			safeGradient.Color = onGradient
			safeModeToggle.BackgroundColor3 = newColor
		end
		if delayGradient and extraDelayEnabled then
			delayGradient.Color = onGradient
			delayToggle.BackgroundColor3 = newColor
		end

		-- Update status text color
		if status then
			status.TextColor3 = newColor
		end
		if execStatus and hasExecutor then
			execStatus.TextColor3 = newColor
		end
	end

	-- Parse hex and update theme color
	hexBox.FocusLost:Connect(function(enterPressed)
		local hex = hexBox.Text:gsub("#", "")
		if #hex == 6 then
			local r = tonumber(hex:sub(1, 2), 16)
			local g = tonumber(hex:sub(3, 4), 16)
			local b = tonumber(hex:sub(5, 6), 16)
			if r and g and b then
				updateThemeColor(Color3.fromRGB(r, g, b))
				colorPreview.BackgroundColor3 = Color3.fromRGB(r, g, b)
			end
		end
	end)

	-- Simple RGB sliders (in a do block to limit local scope)
	do
		local function makeSlider(name, order, defVal, col, onChanged)
			local fr = Instance.new("Frame")
			fr.LayoutOrder = order
			fr.Size = UDim2.new(1, 0, 0, 22)
			fr.BackgroundTransparency = 1
			fr.Parent = miscContent

			local lb = Instance.new("TextLabel")
			lb.Size = UDim2.new(0, 16, 0, 16)
			lb.Position = UDim2.new(0, 0, 0, 3)
			lb.BackgroundTransparency = 1
			lb.Text = name
			lb.TextColor3 = col
			lb.Font = Enum.Font.GothamBold
			lb.TextSize = 11
			lb.TextXAlignment = Enum.TextXAlignment.Left
			lb.Parent = fr

			local bg = Instance.new("Frame")
			bg.Size = UDim2.new(1, -65, 0, 6)
			bg.Position = UDim2.new(0, 20, 0, 8)
			bg.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
			bg.Parent = fr
			Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 3)

			local fill = Instance.new("Frame")
			fill.Size = UDim2.new(defVal / 255, 0, 1, 0)
			fill.BackgroundColor3 = col
			fill.Parent = bg
			Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 3)

			local valLbl = Instance.new("TextLabel")
			valLbl.Size = UDim2.new(0, 35, 0, 16)
			valLbl.Position = UDim2.new(1, -35, 0, 3)
			valLbl.BackgroundTransparency = 1
			valLbl.Text = tostring(defVal)
			valLbl.TextColor3 = Color3.fromRGB(180, 180, 190)
			valLbl.Font = Enum.Font.GothamMedium
			valLbl.TextSize = 10
			valLbl.TextXAlignment = Enum.TextXAlignment.Right
			valLbl.Parent = fr

			local val = defVal
			local btn = Instance.new("TextButton")
			btn.Size = UDim2.new(1, 0, 1, 0)
			btn.BackgroundTransparency = 1
			btn.Text = ""
			btn.Parent = bg

			local dragging = false
			btn.MouseButton1Down:Connect(function() dragging = true end)
			game:GetService("UserInputService").InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
			game:GetService("UserInputService").InputChanged:Connect(function(i)
				if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
					local rel = math.clamp((i.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
					val = math.floor(rel * 255)
					fill.Size = UDim2.new(rel, 0, 1, 0)
					valLbl.Text = tostring(val)
					onChanged(val)
				end
			end)
			btn.MouseButton1Click:Connect(function()
				local m = game:GetService("UserInputService"):GetMouseLocation()
				local rel = math.clamp((m.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
				val = math.floor(rel * 255)
				fill.Size = UDim2.new(rel, 0, 1, 0)
				valLbl.Text = tostring(val)
				onChanged(val)
			end)
			return {getValue = function() return val end, setValue = function(v) val = v; fill.Size = UDim2.new(v/255, 0, 1, 0); valLbl.Text = tostring(v) end}
		end

		local rv, gv, bv = 25, 60, 180
		local rS, gS, bS
		local function updateFromSliders()
			local c = Color3.fromRGB(rv, gv, bv)
			updateThemeColor(c)
			hexBox.Text = string.format("#%02X%02X%02X", rv, gv, bv)
		end
		rS = makeSlider("R", 5, rv, Color3.fromRGB(255, 100, 100), function(v) rv = v; updateFromSliders() end)
		gS = makeSlider("G", 6, gv, Color3.fromRGB(100, 255, 100), function(v) gv = v; updateFromSliders() end)
		bS = makeSlider("B", 7, bv, Color3.fromRGB(100, 100, 255), function(v) bv = v; updateFromSliders() end)
	end

	-- ============================================
	-- TEXT COLOR SECTION (with sliders)
	-- ============================================
	do
		local lbl = Instance.new("TextLabel")
		lbl.LayoutOrder = 8
		lbl.Size = UDim2.new(1, 0, 0, 18)
		lbl.BackgroundTransparency = 1
		lbl.Text = "â€¢ Text Color"
		lbl.TextColor3 = Color3.fromRGB(200, 200, 210)
		lbl.Font = Enum.Font.GothamSemibold
		lbl.TextSize = 13
		lbl.TextXAlignment = Enum.TextXAlignment.Left
		lbl.Parent = miscContent

		local row = Instance.new("Frame")
		row.LayoutOrder = 9
		row.Size = UDim2.new(1, 0, 0, 40)
		row.BackgroundTransparency = 1
		row.Parent = miscContent

		local preview = Instance.new("Frame")
		preview.Size = UDim2.new(0, 40, 0, 40)
		preview.BackgroundColor3 = Color3.fromRGB(200, 200, 210)
		preview.Parent = row
		Instance.new("UICorner", preview).CornerRadius = UDim.new(0, 8)
		Instance.new("UIStroke", preview).Color = Color3.fromRGB(100, 100, 120)

		local container = Instance.new("Frame")
		container.Size = UDim2.new(1, -54, 0, 40)
		container.Position = UDim2.new(0, 50, 0, 0)
		container.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
		container.Parent = row
		Instance.new("UICorner", container).CornerRadius = UDim.new(0, 8)

		local box = Instance.new("TextBox")
		box.Size = UDim2.new(1, -24, 1, 0)
		box.Position = UDim2.new(0, 12, 0, 0)
		box.BackgroundTransparency = 1
		box.TextColor3 = Color3.fromRGB(240, 240, 245)
		box.PlaceholderText = "#C8C8D2"
		box.Font = Enum.Font.GothamMedium
		box.TextSize = 14
		box.Text = "#C8C8D2"
		box.ClearTextOnFocus = false
		box.Parent = container

		local function updateTextColor(c)
			preview.BackgroundColor3 = c
			for _, content in pairs(tabContents) do
				for _, desc in ipairs(content:GetDescendants()) do
					if desc:IsA("TextLabel") and desc.TextSize == 13 then
						desc.TextColor3 = c
					end
				end
			end
		end

		box.FocusLost:Connect(function()
			local hex = box.Text:gsub("#", "")
			if #hex == 6 then
				local r, g, b = tonumber(hex:sub(1,2), 16), tonumber(hex:sub(3,4), 16), tonumber(hex:sub(5,6), 16)
				if r and g and b then updateTextColor(Color3.fromRGB(r, g, b)) end
			end
		end)

		-- Text color sliders
		local function makeSlider2(name, order, defVal, col, onChanged)
			local fr = Instance.new("Frame")
			fr.LayoutOrder = order
			fr.Size = UDim2.new(1, 0, 0, 22)
			fr.BackgroundTransparency = 1
			fr.Parent = miscContent

			local lb = Instance.new("TextLabel")
			lb.Size, lb.Position = UDim2.new(0, 16, 0, 16), UDim2.new(0, 0, 0, 3)
			lb.BackgroundTransparency, lb.Text, lb.TextColor3 = 1, name, col
			lb.Font, lb.TextSize, lb.TextXAlignment = Enum.Font.GothamBold, 11, Enum.TextXAlignment.Left
			lb.Parent = fr

			local bg = Instance.new("Frame")
			bg.Size, bg.Position = UDim2.new(1, -65, 0, 6), UDim2.new(0, 20, 0, 8)
			bg.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
			bg.Parent = fr
			Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 3)

			local fill = Instance.new("Frame")
			fill.Size, fill.BackgroundColor3 = UDim2.new(defVal / 255, 0, 1, 0), col
			fill.Parent = bg
			Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 3)

			local valLbl = Instance.new("TextLabel")
			valLbl.Size, valLbl.Position = UDim2.new(0, 35, 0, 16), UDim2.new(1, -35, 0, 3)
			valLbl.BackgroundTransparency, valLbl.Text = 1, tostring(defVal)
			valLbl.TextColor3, valLbl.Font, valLbl.TextSize = Color3.fromRGB(180, 180, 190), Enum.Font.GothamMedium, 10
			valLbl.TextXAlignment = Enum.TextXAlignment.Right
			valLbl.Parent = fr

			local val, dragging = defVal, false
			local btn = Instance.new("TextButton")
			btn.Size, btn.BackgroundTransparency, btn.Text = UDim2.new(1, 0, 1, 0), 1, ""
			btn.Parent = bg

			btn.MouseButton1Down:Connect(function() dragging = true end)
			game:GetService("UserInputService").InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
			game:GetService("UserInputService").InputChanged:Connect(function(i)
				if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
					local rel = math.clamp((i.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
					val = math.floor(rel * 255)
					fill.Size = UDim2.new(rel, 0, 1, 0)
					valLbl.Text = tostring(val)
					onChanged(val)
				end
			end)
			btn.MouseButton1Click:Connect(function()
				local m = game:GetService("UserInputService"):GetMouseLocation()
				local rel = math.clamp((m.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
				val = math.floor(rel * 255)
				fill.Size = UDim2.new(rel, 0, 1, 0)
				valLbl.Text = tostring(val)
				onChanged(val)
			end)
			return {getValue = function() return val end}
		end

		local tr, tg, tb = 200, 200, 210
		local function updateFromTextSliders()
			local c = Color3.fromRGB(tr, tg, tb)
			updateTextColor(c)
			box.Text = string.format("#%02X%02X%02X", tr, tg, tb)
		end
		makeSlider2("R", 10, tr, Color3.fromRGB(255, 100, 100), function(v) tr = v; updateFromTextSliders() end)
		makeSlider2("G", 11, tg, Color3.fromRGB(100, 255, 100), function(v) tg = v; updateFromTextSliders() end)
		makeSlider2("B", 12, tb, Color3.fromRGB(100, 100, 255), function(v) tb = v; updateFromTextSliders() end)
	end

	-- ============================================
	-- ACCENT COLOR SECTION (with sliders)
	-- ============================================
	do
		local lbl = Instance.new("TextLabel")
		lbl.LayoutOrder = 13
		lbl.Size = UDim2.new(1, 0, 0, 18)
		lbl.BackgroundTransparency = 1
		lbl.Text = "â€¢ Accent Color"
		lbl.TextColor3 = Color3.fromRGB(200, 200, 210)
		lbl.Font = Enum.Font.GothamSemibold
		lbl.TextSize = 13
		lbl.TextXAlignment = Enum.TextXAlignment.Left
		lbl.Parent = miscContent

		local row = Instance.new("Frame")
		row.LayoutOrder = 14
		row.Size = UDim2.new(1, 0, 0, 40)
		row.BackgroundTransparency = 1
		row.Parent = miscContent

		local preview = Instance.new("Frame")
		preview.Size = UDim2.new(0, 40, 0, 40)
		preview.BackgroundColor3 = Color3.fromRGB(65, 65, 85)
		preview.Parent = row
		Instance.new("UICorner", preview).CornerRadius = UDim.new(0, 8)
		Instance.new("UIStroke", preview).Color = Color3.fromRGB(100, 100, 120)

		local container = Instance.new("Frame")
		container.Size = UDim2.new(1, -54, 0, 40)
		container.Position = UDim2.new(0, 50, 0, 0)
		container.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
		container.Parent = row
		Instance.new("UICorner", container).CornerRadius = UDim.new(0, 8)

		local box = Instance.new("TextBox")
		box.Size = UDim2.new(1, -24, 1, 0)
		box.Position = UDim2.new(0, 12, 0, 0)
		box.BackgroundTransparency = 1
		box.TextColor3 = Color3.fromRGB(240, 240, 245)
		box.PlaceholderText = "#414155"
		box.Font = Enum.Font.GothamMedium
		box.TextSize = 14
		box.Text = "#414155"
		box.ClearTextOnFocus = false
		box.Parent = container

		local function updateAccentColor(c)
			preview.BackgroundColor3 = c
			if stroke then stroke.Color = c end
			for _, content in pairs(tabContents) do
				for _, desc in ipairs(content:GetDescendants()) do
					if desc:IsA("UIStroke") and desc.Thickness == 1 then
						desc.Color = c
					end
				end
			end
		end

		box.FocusLost:Connect(function()
			local hex = box.Text:gsub("#", "")
			if #hex == 6 then
				local r, g, b = tonumber(hex:sub(1,2), 16), tonumber(hex:sub(3,4), 16), tonumber(hex:sub(5,6), 16)
				if r and g and b then updateAccentColor(Color3.fromRGB(r, g, b)) end
			end
		end)

		-- Accent color sliders
		local function makeSlider3(name, order, defVal, col, onChanged)
			local fr = Instance.new("Frame")
			fr.LayoutOrder = order
			fr.Size = UDim2.new(1, 0, 0, 22)
			fr.BackgroundTransparency = 1
			fr.Parent = miscContent

			local lb = Instance.new("TextLabel")
			lb.Size, lb.Position = UDim2.new(0, 16, 0, 16), UDim2.new(0, 0, 0, 3)
			lb.BackgroundTransparency, lb.Text, lb.TextColor3 = 1, name, col
			lb.Font, lb.TextSize, lb.TextXAlignment = Enum.Font.GothamBold, 11, Enum.TextXAlignment.Left
			lb.Parent = fr

			local bg = Instance.new("Frame")
			bg.Size, bg.Position = UDim2.new(1, -65, 0, 6), UDim2.new(0, 20, 0, 8)
			bg.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
			bg.Parent = fr
			Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 3)

			local fill = Instance.new("Frame")
			fill.Size, fill.BackgroundColor3 = UDim2.new(defVal / 255, 0, 1, 0), col
			fill.Parent = bg
			Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 3)

			local valLbl = Instance.new("TextLabel")
			valLbl.Size, valLbl.Position = UDim2.new(0, 35, 0, 16), UDim2.new(1, -35, 0, 3)
			valLbl.BackgroundTransparency, valLbl.Text = 1, tostring(defVal)
			valLbl.TextColor3, valLbl.Font, valLbl.TextSize = Color3.fromRGB(180, 180, 190), Enum.Font.GothamMedium, 10
			valLbl.TextXAlignment = Enum.TextXAlignment.Right
			valLbl.Parent = fr

			local val, dragging = defVal, false
			local btn = Instance.new("TextButton")
			btn.Size, btn.BackgroundTransparency, btn.Text = UDim2.new(1, 0, 1, 0), 1, ""
			btn.Parent = bg

			btn.MouseButton1Down:Connect(function() dragging = true end)
			game:GetService("UserInputService").InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
			game:GetService("UserInputService").InputChanged:Connect(function(i)
				if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
					local rel = math.clamp((i.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
					val = math.floor(rel * 255)
					fill.Size = UDim2.new(rel, 0, 1, 0)
					valLbl.Text = tostring(val)
					onChanged(val)
				end
			end)
			btn.MouseButton1Click:Connect(function()
				local m = game:GetService("UserInputService"):GetMouseLocation()
				local rel = math.clamp((m.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
				val = math.floor(rel * 255)
				fill.Size = UDim2.new(rel, 0, 1, 0)
				valLbl.Text = tostring(val)
				onChanged(val)
			end)
			return {getValue = function() return val end}
		end

		local ar, ag, ab = 65, 65, 85
		local function updateFromAccentSliders()
			local c = Color3.fromRGB(ar, ag, ab)
			updateAccentColor(c)
			box.Text = string.format("#%02X%02X%02X", ar, ag, ab)
		end
		makeSlider3("R", 15, ar, Color3.fromRGB(255, 100, 100), function(v) ar = v; updateFromAccentSliders() end)
		makeSlider3("G", 16, ag, Color3.fromRGB(100, 255, 100), function(v) ag = v; updateFromAccentSliders() end)
		makeSlider3("B", 17, ab, Color3.fromRGB(100, 100, 255), function(v) ab = v; updateFromAccentSliders() end)
	end

	-- Enhanced Draggable
	local dragging = false
	local dragStart: Vector2
	local startPos: UDim2
	local dragConnection
	local changeConnection

	topbar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position

			-- Subtle feedback when dragging starts
			local tween = game:GetService("TweenService"):Create(frame, TweenInfo.new(0.1), {BackgroundTransparency = 0.05})
			tween:Play()
		end
	end)

	changeConnection = topbar.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local delta = input.Position - dragStart
			local newPos = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y
			)
			frame.Position = newPos
		end
	end)

	dragConnection = UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
			-- Reset transparency
			local tween = game:GetService("TweenService"):Create(frame, TweenInfo.new(0.1), {BackgroundTransparency = 0})
			tween:Play()
		end
	end)

	-- Enhanced Minimize
	local minimized = false
	-- Store the EXACT original size (420x560)
	local originalWidth = 420
	local originalHeight = 620
	local topbarHeight = 50

	local function setMinimized(on: boolean)
		minimized = on
		
		-- Hide body BEFORE animation when minimizing
		if on then
			body.Visible = false
			minimizeBtn.Text = "+"
		end

		-- Use fixed sizes to prevent drift
		local targetSize = on and UDim2.new(0, originalWidth, 0, topbarHeight) or UDim2.new(0, originalWidth, 0, originalHeight)

		local tween = game:GetService("TweenService"):Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = targetSize})
		tween:Play()

		-- Show body AFTER animation when expanding
		if not on then
			task.delay(0.1, function()
				body.Visible = true
				minimizeBtn.Text = "-"
			end)
		end
	end

	minimizeBtn.MouseButton1Click:Connect(function()
		setMinimized(not minimized)
	end)

	-- Track UI visibility state (used by both close button and Ctrl+L toggle)
	local uiVisible = true

	-- Enhanced Close (hides instead of destroying, can be reopened with Ctrl+L)
	closeBtn.MouseButton1Click:Connect(function()
		-- Simple hide - no transparency animation to avoid breaking UI
		frame.Visible = false
		uiVisible = false
	end)

	-- Enhanced customizable toggle - simple show/hide without transparency issues
	local toggleDebounce = false

	UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
		if gameProcessedEvent then return end
		if waitingForKeybind then return end -- Don't toggle while setting keybind
		
		-- Check if the pressed key matches our keybind
		if input.KeyCode == currentKeybind.key and not toggleDebounce then
			local uis = game:GetService("UserInputService")
			local ctrlHeld = uis:IsKeyDown(Enum.KeyCode.LeftControl) or uis:IsKeyDown(Enum.KeyCode.RightControl)
			local shiftHeld = uis:IsKeyDown(Enum.KeyCode.LeftShift) or uis:IsKeyDown(Enum.KeyCode.RightShift)
			local altHeld = uis:IsKeyDown(Enum.KeyCode.LeftAlt) or uis:IsKeyDown(Enum.KeyCode.RightAlt)
			
			-- Check if modifier requirements match
			if ctrlHeld == currentKeybind.ctrl and shiftHeld == currentKeybind.shift and altHeld == currentKeybind.alt then
				toggleDebounce = true
				uiVisible = not uiVisible
				frame.Visible = uiVisible
				
				task.delay(0.2, function()
					toggleDebounce = false
				end)
			end
		end
	end)


	-- Viewport scaling - update original dimensions after scaling
	local viewportSize = workspace.CurrentCamera.ViewportSize
	local scaleFactor = math.min(viewportSize.X / 1920, viewportSize.Y / 1080)

	if scaleFactor < 1 then
		local newWidth = math.max(380, originalWidth * scaleFactor)
		local newHeight = math.max(420, originalHeight * scaleFactor)
		local newSize = UDim2.new(0, newWidth, 0, newHeight)
		local tween = game:GetService("TweenService"):Create(frame, TweenInfo.new(0.5), {Size = newSize})
		tween:Play()
		-- Update the stored dimensions so minimize/restore works correctly
		originalWidth = newWidth
		originalHeight = newHeight
	end

	return screenGui
end

local gui = createGUI()

-- Set up periodic cleanup
task.spawn(function()
	while true do
		task.wait(300)
		cleanupExpiredCache()
		log(LOG_LEVEL.DEBUG, "Performed periodic cache cleanup")
	end
end)

-- Clean up when players leave
Players.PlayerRemoving:Connect(function(player)
	local userId = player.UserId
	cleanupPlayerConnections(userId)

	-- Clean up cached data for this player
	local userInfoKey = getCacheKey(userId, "userinfo")
	local avatarKey = getCacheKey(userId, "avatarmodel")
	local humDescKey = getCacheKey(userId, "humdesc")

	userInfoCache[userInfoKey] = nil
	avatarModelCache[avatarKey] = nil
	humanoidDescCache[humDescKey] = nil

	log(LOG_LEVEL.DEBUG, "Cleaned up data for player %s (UserId: %d)", player.Name, userId)
end)

-- Print initialization info

if not hasExecutor then
	print("Might wanna get a exec bro")
end

-- Check for potential game compatibility issues
local gameId = game.GameId
local placeId = game.PlaceId
log(LOG_LEVEL.INFO, "Game ID: %d, Place ID: %d", gameId, placeId)

-- Warn about games that might have custom respawn logic
if placeId == 0 then
	log(LOG_LEVEL.WARN, "Running in Roblox Studio - avatar features may not work properly")
elseif gameId == 0 then
	log(LOG_LEVEL.WARN, "Unknown game detected - respawn issues may occur")
end

log(LOG_LEVEL.INFO, "Script initialized successfully with memory management and respawn protection")

