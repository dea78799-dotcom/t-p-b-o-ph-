local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- ID Game
local WORLD_1_ID = 75568173037446
local WORLD_2_ID = 79162127792867

local currentPlaceId = game.PlaceId

-- Kiểm tra ID Game hợp lệ
if currentPlaceId ~= WORLD_1_ID and currentPlaceId ~= WORLD_2_ID then
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Lỗi Game ID!",
        Text = "Script không hỗ trợ Game ID hiện tại: " .. tostring(currentPlaceId),
        Duration = 5
    })
    return
end

local isWorld1 = (currentPlaceId == WORLD_1_ID)

local Window = Rayfield:CreateWindow({
   Name = "Tập Béo Phì " .. (isWorld1 and "(World 1 - Full)" or "(World 2 - Lite)"),
   LoadingTitle = "Đang tải script...",
   LoadingSubtitle = "Tập Béo Phì Hub",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "TapBeoPhiHub",
      FileName = "Settings_" .. tostring(currentPlaceId)
   },
   Discord = { Enabled = false },
   KeySystem = false
})

-- Services & Remote References
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local VirtualUser = game:GetService("VirtualUser")
local Lighting = game:GetService("Lighting")

local Remote = ReplicatedStorage:WaitForChild("Remote", 5)
local Event = Remote and Remote:WaitForChild("Event", 5)
local Function = Remote and Remote:WaitForChild("Function", 5)

-- Variables
local autoTrainEnabled = false
local autoClaimOfflineEnabled = false
local autoClaimOnlineEnabled = false
local autoClaimDailyEnabled = false
local autoRebirthEnabled = false
local autoSpinEnabled = false
local autoCoinEnabled = false
local autoFarmMoneyW2Enabled = false
local autoBuyFoodEnabled = false

local speedEnabled = false
local jumpEnabled = false
local speedValue = 16
local jumpValue = 50
local reportText = ""

-- Variables cho tính năng mới
local antiAFKEnabled = false
local antiAFKConn = nil
local fullbrightEnabled = false

-- Danh sách thức ăn
local foodList = {
   { name = "Cookies", min = 0, max = 110 },
   { name = "Bread slices", min = 110, max = 500 },
   { name = "Gummy bears", min = 500, max = 1500 },
   { name = "Fries", min = 1500, max = 5800 },
   { name = "Donut", min = 5800, max = 15000 },
   { name = "Tucker", min = 15000, max = 40000 },
   { name = "Hamburger", min = 40000, max = 200000 },
   { name = "Mexican Chicken Wrap", min = 200000, max = 750000 },
   { name = "Pizza", min = 750000, max = 3200000 },
   { name = "Cake", min = 3200000, max = 8800000 },
   { name = "Steak", min = 8800000, max = 28500000 },
   { name = "Cream puffs", min = 28500000, max = 345000000 },
   { name = "Pudding", min = 345000000, max = 765000000 },
   { name = "Candied hawthorn", min = 765000000, max = 1350000000 },
   { name = "Pan-fried dumplings", min = 1350000000, max = 3780000000 },
   { name = "Sushi rolls", min = 3780000000, max = 10600000000 },
   { name = "Grilled skewers", min = 10600000000, max = 31200000000 },
   { name = "Fish strips", min = 31200000000, max = 102000000000 },
   { name = "Apple Pie", min = 102000000000, max = 480000000000 },
   { name = "Abalone", min = 480000000000, max = 888000000000 },
}

-- Hàm hiển thị thông báo
local function notify(title, content)
   Rayfield:Notify({
      Title = title,
      Content = content,
      Duration = 3,
      Image = 4483362458,
   })
end

-- HÀM THỰC THI THÔNG THƯỜNG CÓ BẮT LỖI (SAFE CALL)
local function safeCall(featureName, actionFunc, successMsg)
   local success, err = pcall(actionFunc)
   if success then
      if successMsg then
         notify(featureName, successMsg)
      end
   else
      notify("⚠️ " .. featureName .. " Lỗi!", "Tính năng bị lỗi hoặc Admin đã sửa: " .. tostring(err))
   end
   return success, err
end

-- HOOK METAMETHOD ĐỂ BẮT LÚC GAME CHẠY PlayerEndRace
local raceFiredByGame = false
pcall(function()
    if hookmetamethod and Event and Event:FindFirstChild("Race") then
        local PlayerEndRace = Event.Race:FindFirstChild("PlayerEndRace")
        if PlayerEndRace then
            local oldNamecall
            oldNamecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
                local method = getnamecallmethod()
                if not checkcaller() and self == PlayerEndRace and (method == "FireServer" or method == "fireServer") then
                    raceFiredByGame = true
                end
                return oldNamecall(self, ...)
            end))
        end
    end
end)

-- HÀM LẤY MONEY
local function FindMoneyInValue(obj)
    if obj:IsA("ValueBase") then
        local nameLower = string.lower(obj.Name)
        if (string.find(nameLower, "money") or string.find(nameLower, "coin") or string.find(nameLower, "cash")) and not string.find(nameLower, "win") then
            return obj.Value
        end
    end
    return nil
end

local function GetMoneyUniversal()
    if LocalPlayer then
        for _, child in ipairs(LocalPlayer:GetDescendants()) do
            local val = FindMoneyInValue(child)
            if val ~= nil then return val end
        end
    end

    if LocalPlayer and LocalPlayer.Character then
        for _, child in ipairs(LocalPlayer.Character:GetDescendants()) do
            local val = FindMoneyInValue(child)
            if val ~= nil then return val end
        end
    end

    if LocalPlayer then
        for _, child in ipairs(ReplicatedStorage:GetDescendants()) do
            if child.Name == LocalPlayer.Name or string.find(child.Name, tostring(LocalPlayer.UserId)) then
                for _, subChild in ipairs(child:GetDescendants()) do
                    local val = FindMoneyInValue(subChild)
                    if val ~= nil then return val end
                end
            end
        end
    end

    local playerGui = LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")
    if playerGui then
        local candidates = {}
        for _, obj in ipairs(playerGui:GetDescendants()) do
            if obj:IsA("TextLabel") and obj.Visible then
                local txt = obj.Text
                if not string.find(txt, "/") and not string.find(string.lower(txt), "win") then
                    if string.match(txt, "%d+%.?%d*[MKBThmkbt]") or string.match(txt, "%$%d+") then
                        if obj.AbsolutePosition.Y < 120 and obj.AbsolutePosition.X > 300 then
                            table.insert(candidates, {label = obj, x = obj.AbsolutePosition.X})
                        end
                    end
                end
            end
        end

        if #candidates > 0 then
            table.sort(candidates, function(a, b) return a.x > b.x end)
            return candidates[1].label.Text
        end
    end

    return nil
end

local function ParseMoney(val)
    if type(val) == "number" then return val end
    if type(val) ~= "string" then return 0 end
    
    local cleanStr = string.gsub(val, "[%$,%s]", "")
    local num, suffix = string.match(cleanStr, "([%d%.]+)(%a?)")
    num = tonumber(num) or 0
    
    if suffix then
        suffix = string.lower(suffix)
        if suffix == "k" then num = num * 1e3
        elseif suffix == "m" then num = num * 1e6
        elseif suffix == "b" then num = num * 1e9
        elseif suffix == "t" then num = num * 1e12
        end
    end
    return num
end

-- Server Hop
local function JoinLowServer()
   notify("Server Hop", "Đang tìm server dưới 3 người...")
   safeCall("Server Hop", function()
      local placeId = game.PlaceId
      local cursor = ""
      local foundServer = nil

      repeat
         local url = "https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100" .. (cursor ~= "" and "&cursor=" .. cursor or "")
         local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
         end)

         if success and result and result.data then
            cursor = result.cursor or ""
            for _, server in ipairs(result.data) do
               if server.playing < 3 and server.id ~= game.JobId then
                  foundServer = server.id
                  break
               end
            end
         else
            break
         end
         task.wait(0.2)
      until foundServer or cursor == ""

      if foundServer then
         notify("Server Hop", "Đã tìm thấy! Đang chuyển server...")
         TeleportService:TeleportToPlaceInstance(placeId, foundServer, LocalPlayer)
      else
         notify("Server Hop", "Không tìm thấy server phù hợp!")
      end
   end)
end

---------------------------------------------------------
-- TAB FARM
---------------------------------------------------------
local FarmTab = Window:CreateTab("Farm", 4483362458)

FarmTab:CreateButton({
   Name = "Vào server ít người (<3 người)",
   Callback = function()
      JoinLowServer()
   end,
})

FarmTab:CreateButton({
   Name = "Trang bị thức ăn",
   Callback = function()
      safeCall("Trang bị thức ăn", function()
         Event.Eat.PlayerIsEat:FireServer()
      end, "Đã gửi yêu cầu trang bị thức ăn!")
   end,
})

-- TỰ ĐỘNG TẬP (35 LẦN / GIÂY)
FarmTab:CreateToggle({
   Name = "Tự động tập",
   CurrentValue = false,
   Flag = "AutoTrainToggle",
   Callback = function(Value)
      autoTrainEnabled = Value
      if autoTrainEnabled then
         notify("Tự động tập", "Trạng thái: BẬT (35 lần/giây)")
         pcall(function() Event.Eat.PlayerIsEat:FireServer() end)
         
         task.spawn(function()
            while autoTrainEnabled do
               local success, err = pcall(function()
                  Event.Eat.PlayerTryClickRE:FireServer(true)
               end)
               if not success then
                  notify("⚠️ Tự động tập bị lỗi!", "Lỗi từ game: " .. tostring(err))
                  autoTrainEnabled = false
                  break
               end
               task.wait(1 / 35)
            end
         end)
      else
         notify("Tự động tập", "Trạng thái: TẮT")
      end
   end,
})

-- AUTO CÀY XU WORLD 1 / WORLD 2
if isWorld1 then
   FarmTab:CreateToggle({
      Name = "Auto cày xu",
      CurrentValue = false,
      Flag = "AutoCoinToggle",
      Callback = function(Value)
         autoCoinEnabled = Value
         if autoCoinEnabled then
            notify("Auto Cày Xu", "Trạng thái: BẬT (Dịch chuyển W1)")
            
            task.spawn(function()
               local targetCFrame = CFrame.new(0.73, 5004.52, -84.40)
               
               while autoCoinEnabled do
                  local success, err = pcall(function()
                     local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
                     local hrp = character:WaitForChild("HumanoidRootPart", 5)
                     
                     if hrp then
                        hrp.CFrame = targetCFrame
                        local raceEnded = false
                        local eventConn
                        
                        local PlayerEndRace = Event.Race.PlayerEndRace
                        eventConn = PlayerEndRace.OnClientEvent:Connect(function()
                           raceEnded = true
                           if eventConn then eventConn:Disconnect() end
                        end)
                        
                        local timeout = 0
                        while not raceEnded and autoCoinEnabled and timeout < 30 do
                           task.wait(0.5)
                           timeout = timeout + 0.5
                        end
                        
                        if eventConn then eventConn:Disconnect() end
                        if autoCoinEnabled then task.wait(10) end
                     end
                  end)

                  if not success then
                     notify("⚠️ Auto Cày Xu bị lỗi!", tostring(err))
                     autoCoinEnabled = false
                     break
                  end
               end
            end)
         else
            notify("Auto Cày Xu", "Trạng thái: TẮT")
         end
      end,
   })
else
   FarmTab:CreateToggle({
      Name = "Auto farm money",
      CurrentValue = false,
      Flag = "AutoFarmMoneyW2Toggle",
      Callback = function(Value)
         autoFarmMoneyW2Enabled = Value
         if autoFarmMoneyW2Enabled then
            notify("Auto Farm Money", "Trạng thái: BẬT (World 2 - Nghỉ 8s)")
            
            task.spawn(function()
               local targetCFrame = CFrame.new(-4999.87, 8356.70, -179.45)
               local loopCount = 0
               
               while autoFarmMoneyW2Enabled do
                  local success, err = pcall(function()
                     local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
                     local hrp = character:FindFirstChild("HumanoidRootPart")
                     
                     if hrp then
                        raceFiredByGame = false
                        hrp.CFrame = targetCFrame
                        
                        local timeout = 0
                        while not raceFiredByGame and timeout < 15 and autoFarmMoneyW2Enabled do
                           task.wait(0.1)
                           timeout = timeout + 0.1
                        end
                        
                        if autoFarmMoneyW2Enabled then
                           loopCount = loopCount + 1
                           notify("Auto Farm Money W2", "Lần " .. tostring(loopCount) .. ": Game đã tự gửi xong! Chờ 8s...")
                           task.wait(8)
                        end
                     else
                        task.wait(1)
                     end
                  end)

                  if not success then
                     notify("⚠️ Auto Farm Money bị lỗi!", tostring(err))
                     autoFarmMoneyW2Enabled = false
                     break
                  end
               end
            end)
         else
            notify("Auto Farm Money", "Trạng thái: TẮT")
         end
      end,
   })
end

FarmTab:CreateToggle({
   Name = "Tự động nhận chất béo khi online",
   CurrentValue = false,
   Flag = "AutoClaimOfflineToggle",
   Callback = function(Value)
      autoClaimOfflineEnabled = Value
      if autoClaimOfflineEnabled then
         notify("Tự nhận chất béo", "Trạng thái: BẬT (Mỗi 10 giây)")
         task.spawn(function()
            while autoClaimOfflineEnabled do
               local success, err = pcall(function()
                  Event.Offline.ClaimOfflineReward:FireServer()
               end)
               if not success then
                  notify("⚠️ Tự nhận chất béo lỗi!", tostring(err))
                  autoClaimOfflineEnabled = false
                  break
               end
               task.wait(10)
            end
         end)
      else
         notify("Tự nhận chất béo", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateToggle({
   Name = "Nhận quà trực tuyến",
   CurrentValue = false,
   Flag = "AutoClaimOnlineToggle",
   Callback = function(Value)
      autoClaimOnlineEnabled = Value
      if autoClaimOnlineEnabled then
         notify("Quà trực tuyến", "Trạng thái: BẬT (Tự động nhận 1-20)")
         task.spawn(function()
            while autoClaimOnlineEnabled do
               local success, err = pcall(function()
                  for i = 1, 20 do
                     if not autoClaimOnlineEnabled then break end
                     Event.Reward["[C-S]TryGetReward"]:FireServer(tostring(i))
                     task.wait(1)
                  end
               end)
               if not success then
                  notify("⚠️ Nhận quà trực tuyến lỗi!", tostring(err))
                  autoClaimOnlineEnabled = false
                  break
               end
               task.wait(5)
            end
         end)
      else
         notify("Quà trực tuyến", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateToggle({
   Name = "Nhận quà hàng ngày",
   CurrentValue = false,
   Flag = "AutoClaimDailyToggle",
   Callback = function(Value)
      autoClaimDailyEnabled = Value
      if autoClaimDailyEnabled then
         notify("Quà hàng ngày", "Trạng thái: BẬT (Tự động nhận)")
         task.spawn(function()
            while autoClaimDailyEnabled do
               local success, err = pcall(function()
                  for i = 1, 10 do
                     if not autoClaimDailyEnabled then break end
                     Event.DailyPack.TryClaimDailyPackReward:FireServer(i)
                     task.wait(0.1)
                  end
               end)
               if not success then
                  notify("⚠️ Nhận quà hàng ngày lỗi!", tostring(err))
                  autoClaimDailyEnabled = false
                  break
               end
               task.wait(5)
            end
         end)
      else
         notify("Quà hàng ngày", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateButton({
   Name = "Nhận quà hàng ngày (2)",
   Callback = function()
      safeCall("Quà hàng ngày (2)", function()
         Function.DailySign.canClaim:InvokeServer()
      end, "Đã gửi yêu cầu nhận quà hàng ngày (2)!")
   end,
})

FarmTab:CreateToggle({
   Name = "Tự động tái sinh",
   CurrentValue = false,
   Flag = "AutoRebirthToggle",
   Callback = function(Value)
      autoRebirthEnabled = Value
      if autoRebirthEnabled then
         notify("Tự động tái sinh", "Trạng thái: BẬT (Mỗi 10 giây)")
         task.spawn(function()
            while autoRebirthEnabled do
               local success, err = pcall(function()
                  Event.Rebirth.TryRebirth:FireServer()
               end)
               if not success then
                  notify("⚠️ Tự động tái sinh lỗi!", tostring(err))
                  autoRebirthEnabled = false
                  break
               end
               task.wait(10)
            end
         end)
      else
         notify("Tự động tái sinh", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateButton({
   Name = "Tái sinh ngay bây giờ",
   Callback = function()
      safeCall("Tái sinh", function()
         Event.Rebirth.TryRebirth:FireServer()
      end, "Đã thực hiện tái sinh ngay!")
   end,
})

FarmTab:CreateButton({
   Name = "Quay vòng quay",
   Callback = function()
      safeCall("Vòng quay", function()
         Function.Spin["[C-S]TrySpin"]:InvokeServer()
      end, "Đã thực hiện quay vòng quay!")
   end,
})

FarmTab:CreateToggle({
   Name = "Tự động quay vòng quay",
   CurrentValue = false,
   Flag = "AutoSpinToggle",
   Callback = function(Value)
      autoSpinEnabled = Value
      if autoSpinEnabled then
         notify("Vòng quay", "Trạng thái: BẬT Auto quay (1/10 giây)")
         task.spawn(function()
            while autoSpinEnabled do
               local success, err = pcall(function()
                  Function.Spin["[C-S]TrySpin"]:InvokeServer()
               end)
               if not success then
                  notify("⚠️ Auto Quay Vòng Quay Lỗi!", tostring(err))
                  autoSpinEnabled = false
                  break
               end
               task.wait(1 / 10)
            end
         end)
      else
         notify("Vòng quay", "Trạng thái: TẮT Auto quay")
      end
   end,
})

---------------------------------------------------------
-- TAB PET
---------------------------------------------------------
local PetTab = Window:CreateTab("Pet", 4483362458)

PetTab:CreateButton({
   Name = "Gỡ trang bị pet",
   Callback = function()
      safeCall("Tháo Pet", function()
         Event.Pet.UnEquipAll:FireServer()
      end, "Đã gỡ tất cả pet đang trang bị!")
   end,
})

if isWorld1 then
   PetTab:CreateButton({
      Name = "Random trứng 1",
      Callback = function()
         safeCall("Mở Trứng 1", function()
            Function.Luck["[C-S]DoLuck"]:InvokeServer("Egg1", 1)
         end, "Đã thực hiện mở Trứng 1!")
      end,
   })

   PetTab:CreateButton({
      Name = "Random pet 2",
      Callback = function()
         safeCall("Mở Trứng 2", function()
            Function.Luck["[C-S]DoLuck"]:InvokeServer("Egg2", 1)
         end, "Đã thực hiện mở Trứng 2!")
      end,
   })

   PetTab:CreateButton({
      Name = "Random pet 3",
      Callback = function()
         safeCall("Mở Trứng 3", function()
            Function.Luck["[C-S]DoLuck"]:InvokeServer("Egg3", 1)
         end, "Đã thực hiện mở Trứng 3!")
      end,
   })
else
   PetTab:CreateButton({
      Name = "Random pet 1 (Thế giới 2)",
      Callback = function()
         safeCall("Mở Trứng 1 W2", function()
            Function.Luck["[C-S]DoLuck"]:InvokeServer("Egg1", 1)
         end, "Đã gửi yêu cầu mở Pet 1 Thế giới 2!")
      end,
   })

   PetTab:CreateButton({
      Name = "Random pet 2 (Thế giới 2)",
      Callback = function()
         safeCall("Mở Trứng 2 W2", function()
            Function.Luck["[C-S]DoLuck"]:InvokeServer("Egg2", 1)
         end, "Đã gửi yêu cầu mở Pet 2 Thế giới 2!")
      end,
   })
end

PetTab:CreateButton({
   Name = "Random pet event free",
   Callback = function()
      safeCall("Pet Event Free", function()
         Event.PetEvent.TryOpenEventEgg:FireServer()
      end, "Đã gửi yêu cầu mở Pet Event Free!")
   end,
})

---------------------------------------------------------
-- TAB MUA ĐỒ
---------------------------------------------------------
local ShopTab = Window:CreateTab("Mua đồ", 4483362458)

ShopTab:CreateButton({
   Name = "Về thế giới 1",
   Callback = function()
      safeCall("Chuyển Thế Giới", function()
         Event.World.TryTeleportWorld:FireServer(1)
      end, "Đã chuyển/mở khóa Thế Giới 1")
   end,
})

ShopTab:CreateButton({
   Name = "Mở khóa hoặc tele qua thế giới 2",
   Callback = function()
      safeCall("Chuyển Thế Giới", function()
         Event.World.TryTeleportWorld:FireServer(2)
      end, "Đã chuyển/mở khóa Thế Giới 2")
   end,
})

ShopTab:CreateButton({
   Name = "Mở khóa hoặc tele về thế giới 3",
   Callback = function()
      safeCall("Chuyển Thế Giới", function()
         Event.World.TryTeleportWorld:FireServer(3)
      end, "Đã chuyển/mở khóa Thế Giới 3")
   end,
})

ShopTab:CreateButton({
   Name = "Kiểm tra có bao nhiêu tiền",
   Callback = function()
      safeCall("Kiểm Tra Tiền", function()
         local moneyRaw = GetMoneyUniversal()
         if moneyRaw then
            local moneyNum = ParseMoney(moneyRaw)
            notify("Số tiền hiện tại", "Bạn đang có: " .. tostring(moneyRaw) .. " (" .. tostring(moneyNum) .. ")")
         else
            notify("Số tiền hiện tại", "Không tìm thấy dữ liệu số tiền!")
         end
      end)
   end,
})

ShopTab:CreateToggle({
   Name = "Mua thức ăn tiếp theo",
   CurrentValue = false,
   Flag = "AutoBuyFoodToggle",
   Callback = function(Value)
      autoBuyFoodEnabled = Value
      if autoBuyFoodEnabled then
         notify("Auto Thức Ăn", "Trạng thái: BẬT (Tự động kiểm tra & mua)")
         
         task.spawn(function()
            local lastNotifyTime = 0
            while autoBuyFoodEnabled do
               local success, err = pcall(function()
                  local moneyRaw = GetMoneyUniversal()
                  if moneyRaw then
                     local money = ParseMoney(moneyRaw)
                     if money > 888000000000 then
                        if tick() - lastNotifyTime > 15 then
                           notify("Thông báo", "Chưa được cập nhật hoặc cần qua thế giới mới")
                           lastNotifyTime = tick()
                        end
                     else
                        local targetIndex = nil
                        for i, food in ipairs(foodList) do
                           if money >= food.min and money < food.max then
                              targetIndex = i
                              break
                           end
                        end
                        
                        if targetIndex then
                           for i = targetIndex, 1, -1 do
                              if not autoBuyFoodEnabled then break end
                              Event.Food.TryUnlockFood:FireServer(foodList[i].name)
                              task.wait(0.5)
                           end
                        end
                     end
                  else
                     if tick() - lastNotifyTime > 15 then
                        notify("Auto Thức Ăn", "Không thể lấy số tiền hiện tại!")
                        lastNotifyTime = tick()
                     end
                  end
               end)

               if not success then
                  notify("⚠️ Auto Buy Food Bị Lỗi!", tostring(err))
                  autoBuyFoodEnabled = false
                  break
               end
               task.wait(3)
            end
         end)
      else
         notify("Auto Thức Ăn", "Trạng thái: TẮT")
      end
   end,
})

-- HIỆU ỨNG DI CHUYỂN (TRAIL)
local function buyTrail(name, displayName)
   safeCall("Mua Đường Mòn", function()
      Event.Trail.TryUnlockTrail:FireServer(name)
   end, "Đã gửi yêu cầu mua " .. displayName .. "!")
end

ShopTab:CreateButton({ Name = "Mua đường mòn nước", Callback = function() buyTrail("Blister", "Đường mòn nước") end })
ShopTab:CreateButton({ Name = "Mua đường mòn sấm sét", Callback = function() buyTrail("Lightning", "Đường mòn sấm sét") end })
ShopTab:CreateButton({ Name = "Mua đường mòn nhạc", Callback = function() buyTrail("Music", "Đường mòn nhạc") end })
ShopTab:CreateButton({ Name = "Mua hiệu ứng di chuyển ong", Callback = function() buyTrail("Bee", "Hiệu ứng di chuyển ong") end })
ShopTab:CreateButton({ Name = "Hiệu ứng di chuyển hacker", Callback = function() buyTrail("Hacker", "Hiệu ứng di chuyển Hacker") end })
ShopTab:CreateButton({ Name = "Hiệu ứng di chuyển lá phong", Callback = function() buyTrail("Maple leaves", "Hiệu ứng lá phong") end })
ShopTab:CreateButton({ Name = "Hiệu ứng di chuyển băng", Callback = function() buyTrail("Frost", "Hiệu ứng di chuyển băng") end })
ShopTab:CreateButton({ Name = "Mua hiệu ứng di chuyển lửa", Callback = function() buyTrail("Flame", "Hiệu ứng di chuyển lửa") end })
ShopTab:CreateButton({ Name = "Hiệu ứng di chuyển xương", Callback = function() buyTrail("Bone", "Hiệu ứng di chuyển xương") end })

-- VA CHẠM (IMPACT)
local function buyImpact(name, displayName)
   safeCall("Mua Va Chạm", function()
      Event.Impact.TryUnlockImpact:FireServer(name)
   end, "Đã gửi yêu cầu mua " .. displayName .. "!")
end

ShopTab:CreateButton({ Name = "Hiệu ứng va chạm tia chớp", Callback = function() buyImpact("Lightning", "Va chạm tia chớp") end })
ShopTab:CreateButton({ Name = "Mua va chạm lửa", Callback = function() buyImpact("Flame", "Va chạm lửa") end })
ShopTab:CreateButton({ Name = "Va chạm bụi", Callback = function() buyImpact("Dust", "Va chạm bụi") end })
ShopTab:CreateButton({ Name = "Va chạm nước", Callback = function() buyImpact("Water splash", "Va chạm nước") end })
ShopTab:CreateButton({ Name = "Va chạm bóng ma", Callback = function() buyImpact("Ghosts", "Va chạm bóng ma") end })

---------------------------------------------------------
-- TAB NHẬN HUY HIỆU
---------------------------------------------------------
local AchievementTab = Window:CreateTab("Nhận huy hiệu", 4483362458)

local function claimAchievement(name, displayName)
   safeCall("Huy Hiệu", function()
      Event.Achievement.TryClaimAchievement:FireServer(name)
   end, "Đã gửi yêu cầu nhận huy hiệu " .. displayName .. "!")
end

AchievementTab:CreateButton({ Name = "Nhận huy hiệu tái sinh", Callback = function() claimAchievement("Rebirth", "Tái sinh") end })
AchievementTab:CreateButton({ Name = "Nhận huy hiệu thời gian", Callback = function() claimAchievement("Time", "Thời gian") end })
AchievementTab:CreateButton({ Name = "Nhận huy hiệu mở trứng", Callback = function() claimAchievement("Egg", "Mở trứng") end })
AchievementTab:CreateButton({ Name = "Nhận huy hiệu xé băng keo", Callback = function() claimAchievement("Tape", "Xé băng keo") end })

---------------------------------------------------------
-- TAB THÔNG TIN NGƯỜI CHƠI
---------------------------------------------------------
local PlayerTab = Window:CreateTab("Thông Tin Người Chơi", 4483362458)

-- TÍNH NĂNG MỚI: ANTI AFK
PlayerTab:CreateToggle({
   Name = "Anti AFK",
   CurrentValue = false,
   Flag = "AntiAFKToggle",
   Callback = function(Value)
      antiAFKEnabled = Value
      if antiAFKEnabled then
         notify("Anti AFK", "Trạng thái: BẬT (Đã chống bị kick AFK)")
         antiAFKConn = LocalPlayer.Idled:Connect(function()
            if antiAFKEnabled then
               pcall(function()
                  VirtualUser:CaptureController()
                  VirtualUser:ClickButton2(Vector2.new())
               end)
               notify("Anti AFK", "Đã chặn game đá bạn ra ngoài thành công!")
            end
         end)
      else
         notify("Anti AFK", "Trạng thái: TẮT")
         if antiAFKConn then
            antiAFKConn:Disconnect()
            antiAFKConn = nil
         end
      end
   end,
})

-- TÍNH NĂNG MỚI: PHÁT SÁNG BẢN ĐỒ (FULLBRIGHT)
PlayerTab:CreateToggle({
   Name = "Phát sáng bản đồ",
   CurrentValue = false,
   Flag = "FullbrightToggle",
   Callback = function(Value)
      fullbrightEnabled = Value
      if fullbrightEnabled then
         notify("Phát sáng bản đồ", "Trạng thái: BẬT (Làm sáng mọi vùng tối)")
         task.spawn(function()
            while fullbrightEnabled do
               pcall(function()
                  Lighting.Ambient = Color3.fromRGB(255, 255, 255)
                  Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
                  Lighting.Brightness = 2
                  Lighting.ClockTime = 14
                  Lighting.FogEnd = 1e10
               end)
               task.wait(1)
            end
         end)
      else
         notify("Phát sáng bản đồ", "Trạng thái: TẮT (Trở lại ánh sáng mặc định)")
         pcall(function()
            Lighting.Ambient = Color3.fromRGB(128, 128, 128)
            Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
            Lighting.Brightness = 1
            Lighting.ClockTime = 12
            Lighting.FogEnd = 100000
         end)
      end
   end,
})

-- TÍNH NĂNG MỚI: KIỂM TRA CÁC TÍNH NĂNG ĐANG BẬT
PlayerTab:CreateButton({
   Name = "Kiểm tra tính năng đang bật",
   Callback = function()
      safeCall("Kiểm Tra Tính Năng", function()
         local activeList = {}

         if autoTrainEnabled then table.insert(activeList, "• Tự động tập") end
         if isWorld1 and autoCoinEnabled then table.insert(activeList, "• Auto cày xu (World 1)") end
         if not isWorld1 and autoFarmMoneyW2Enabled then table.insert(activeList, "• Auto farm money (World 2)") end
         if autoClaimOfflineEnabled then table.insert(activeList, "• Tự động nhận chất béo khi online") end
         if autoClaimOnlineEnabled then table.insert(activeList, "• Nhận quà trực tuyến") end
         if autoClaimDailyEnabled then table.insert(activeList, "• Nhận quà hàng ngày") end
         if autoRebirthEnabled then table.insert(activeList, "• Tự động tái sinh") end
         if autoSpinEnabled then table.insert(activeList, "• Tự động quay vòng quay") end
         if autoBuyFoodEnabled then table.insert(activeList, "• Mua thức ăn tiếp theo") end
         if speedEnabled then table.insert(activeList, "• Bật tốc độ (" .. tostring(speedValue) .. ")") end
         if jumpEnabled then table.insert(activeList, "• Bật nhảy cao (" .. tostring(jumpValue) .. ")") end
         if antiAFKEnabled then table.insert(activeList, "• Anti AFK") end
         if fullbrightEnabled then table.insert(activeList, "• Phát sáng bản đồ") end

         if #activeList > 0 then
            notify("Tính năng đang BẬT (" .. #activeList .. ")", table.concat(activeList, "\n"))
         else
            notify("Tính năng đang BẬT", "Hiện tại không có tính năng nào đang bật!")
         end
      end)
   end,
})

PlayerTab:CreateButton({
   Name = 'Nhập code "FAT"',
   Callback = function()
      safeCall("Nhập Code", function()
         ReplicatedStorage.CdkRewardFuntion.isPlayerUseCdkRequest:InvokeServer("FAT")
      end, "Đã gửi yêu cầu nhập code FAT!")
   end,
})

PlayerTab:CreateButton({
   Name = "Reset người chơi",
   Callback = function()
      safeCall("Reset Nhân Vật", function()
         if LocalPlayer.Character then
            LocalPlayer.Character:BreakJoints()
         end
      end, "Đã reset nhân vật thành công!")
   end,
})

PlayerTab:CreateButton({
   Name = "Ragdoll",
   Callback = function()
      safeCall("Ragdoll", function()
         Event.Race.Ragdoll:FireServer()
      end, "Đã gửi yêu cầu Bật Ragdoll!")
   end,
})

PlayerTab:CreateButton({
   Name = "Tắt ragdoll",
   Callback = function()
      safeCall("Tắt Ragdoll", function()
         Event.Race.UnRagdoll:FireServer()
      end, "Đã gửi yêu cầu Tắt Ragdoll!")
   end,
})

PlayerTab:CreateButton({
   Name = "Bật chế độ Giảm Lag (Anti-Lag)",
   Callback = function()
      safeCall("Giảm Lag", function()
         local workspace = game:GetService("Workspace")
         
         Lighting.GlobalShadows = false
         Lighting.FogEnd = 9e9
         for _, v in ipairs(Lighting:GetChildren()) do
            if v:IsA("PostEffect") or v:IsA("Atmosphere") or v:IsA("Sky") then
               v:Destroy()
            end
         end
         
         for _, v in ipairs(workspace:GetDescendants()) do
            if v:IsA("BasePart") and not v:IsA("MeshPart") then
               v.Material = Enum.Material.SmoothPlastic
               v.CastShadow = false
            elseif v:IsA("Decal") or v:IsA("Texture") then
               v:Destroy()
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
               v.Enabled = false
            end
         end
      end, "Đã tối ưu đồ họa thành công!")
   end,
})

PlayerTab:CreateSlider({
   Name = "Tốc độ",
   Range = {1, 500},
   Increment = 1,
   CurrentValue = 16,
   Flag = "SpeedSlider",
   Callback = function(Value)
      speedValue = Value
   end,
})

PlayerTab:CreateToggle({
   Name = "Bật tốc độ",
   CurrentValue = false,
   Flag = "SpeedToggle",
   Callback = function(Value)
      speedEnabled = Value
      if speedEnabled then
         notify("Tốc độ", "Đã BẬT tốc độ chạy: " .. tostring(speedValue))
         task.spawn(function()
            while speedEnabled do
               local success, err = pcall(function()
                  if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                     LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = speedValue
                  end
               end)
               if not success then
                  notify("⚠️ Tốc độ chạy bị lỗi!", tostring(err))
                  speedEnabled = false
                  break
               end
               task.wait(0.1)
            end
            pcall(function()
               if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                  LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
               end
            end)
         end)
      else
         notify("Tốc độ", "Đã TẮT tốc độ chạy (Trở về mặc định)")
      end
   end,
})

PlayerTab:CreateSlider({
   Name = "Nhảy cao",
   Range = {1, 500},
   Increment = 1,
   CurrentValue = 50,
   Flag = "JumpSlider",
   Callback = function(Value)
      jumpValue = Value
   end,
})

PlayerTab:CreateToggle({
   Name = "Bật nhảy cao",
   CurrentValue = false,
   Flag = "JumpToggle",
   Callback = function(Value)
      jumpEnabled = Value
      if jumpEnabled then
         notify("Nhảy cao", "Đã BẬT độ cao nhảy: " .. tostring(jumpValue))
         task.spawn(function()
            while jumpEnabled do
               local success, err = pcall(function()
                  if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                     local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                     hum.UseJumpPower = true
                     hum.JumpPower = jumpValue
                  end
               end)
               if not success then
                  notify("⚠️ Nhảy cao bị lỗi!", tostring(err))
                  jumpEnabled = false
                  break
               end
               task.wait(0.1)
            end
            pcall(function()
               if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                  LocalPlayer.Character:FindFirstChildOfClass("Humanoid").JumpPower = 50
               end
            end)
         end)
      else
         notify("Nhảy cao", "Đã TẮT độ cao nhảy (Trở về mặc định)")
      end
   end,
})

-- SECTION BÁO LỖI / GÓP Ý
PlayerTab:CreateInput({
   Name = "Nội dung báo lỗi / Góp ý",
   PlaceholderText = "Nhập nội dung lỗi hoặc tin nhắn góp ý...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      reportText = Text
   end,
})

PlayerTab:CreateButton({
   Name = "Gửi báo lỗi về Discord",
   Callback = function()
      if reportText == "" or string.gsub(reportText, "%s+", "") == "" then
         notify("Báo Lỗi", "Vui lòng nhập nội dung trước khi gửi!")
         return
      end
      
      safeCall("Gửi Báo Lỗi", function()
         local webhookUrl = "https://discord.com/api/webhooks/1545333668187344957/jWX4F4hfLlZJ6-7uslrSamudPk_FsOQQf6QHcxJGFSbZxlsFZcSFgM5EVdJgSxI8niwy"
         local reqFunc = (syn and syn.request) or (http and http.request) or http_request or request
         
         if not reqFunc then
            error("Executor không hỗ trợ hàm gửi Webhook (http/request missing)!")
         end

         local payload = HttpService:JSONEncode({
            content = "🔔 **Có báo lỗi / góp ý mới!**",
            embeds = {{
               title = "📋 Báo Lỗi / Góp Ý Từ Người Chơi",
               description = reportText,
               color = 16711680,
               fields = {
                  { name = "Tên người chơi", value = LocalPlayer.Name .. " (@" .. LocalPlayer.DisplayName .. ")", inline = true },
                  { name = "User ID", value = tostring(LocalPlayer.UserId), inline = true },
                  { name = "Game ID", value = tostring(game.PlaceId), inline = true }
               },
               footer = { text = "Tập Béo Phì Hub" }
            }}
         })
         
         reqFunc({
            Url = webhookUrl,
            Method = "POST",
            Headers = { ["Content-Type"] = "application/json" },
            Body = payload
         })
      end, "Đã gửi báo lỗi thành công về Discord!")
   end,
})

-- Khôi phục cấu hình riêng cho từng World
Rayfield:LoadConfiguration()
