local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Tập Béo Phì",
   LoadingTitle = "Đang tải script...",
   LoadingSubtitle = "Tập Béo Phì Hub",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "TapBeoPhiHub",
      FileName = "Settings"
   },
   Discord = { Enabled = false },
   KeySystem = false
})

-- Services & Remote References
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remote = ReplicatedStorage:WaitForChild("Remote")
local Event = Remote:WaitForChild("Event")
local Function = Remote:WaitForChild("Function")

local EatEvent = Event:WaitForChild("Eat")
local PlayerIsEat = EatEvent:WaitForChild("PlayerIsEat")
local PlayerTryClickRE = EatEvent:WaitForChild("PlayerTryClickRE")
local PlayerEndRace = Event:WaitForChild("Race"):WaitForChild("PlayerEndRace")
local TryUnlockFood = Event:WaitForChild("Food"):WaitForChild("TryUnlockFood")

-- Variables
local autoTrainEnabled = false
local autoClaimOfflineEnabled = false
local autoRebirthEnabled = false
local autoSpinEnabled = false
local autoCoinEnabled = false
local autoBuyFoodEnabled = false

local speedEnabled = false
local jumpEnabled = false
local speedValue = 16
local jumpValue = 50

-- Danh sách thức ăn xếp theo mức giá (Min - Max)
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
}

-- Hàm hiển thị thông báo bằng Rayfield
local function notify(title, content)
   Rayfield:Notify({
      Title = title,
      Content = content,
      Duration = 3,
      Image = 4483362458,
   })
end

-- =================================-------------------
-- HÀM LẤY MONEY TỪ MỌI NGUỒN VÀ CHUYỂN ĐỔI SỐ
-- =================================-------------------
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
            table.sort(candidates, function(a, b)
                return a.x > b.x
            end)
            return candidates[1].label.Text
        end
    end

    return nil
end

-- Hàm chuyển đổi định dạng chuỗi Money (ví dụ: "3.2M", "500k", "$1,000") về dạng Số (number)
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

---------------------------------------------------------
-- TAB FARM
---------------------------------------------------------
local FarmTab = Window:CreateTab("Farm", 4483362458)

FarmTab:CreateButton({
   Name = "Trang bị thức ăn",
   Callback = function()
      PlayerIsEat:FireServer()
      notify("Farm", "Đã gửi yêu cầu trang bị thức ăn!")
   end,
})

FarmTab:CreateToggle({
   Name = "Tự động tập",
   CurrentValue = false,
   Flag = "AutoTrainToggle",
   Callback = function(Value)
      autoTrainEnabled = Value
      if autoTrainEnabled then
         notify("Tự động tập", "Trạng thái: BẬT (20 lần/giây)")
         PlayerIsEat:FireServer()
         
         task.spawn(function()
            while autoTrainEnabled do
               local args = { true }
               PlayerTryClickRE:FireServer(unpack(args))
               task.wait(1 / 20)
            end
         end)
      else
         notify("Tự động tập", "Trạng thái: TẮT")
      end
   end,
})

FarmTab:CreateToggle({
   Name = "Auto cày xu",
   CurrentValue = false,
   Flag = "AutoCoinToggle",
   Callback = function(Value)
      autoCoinEnabled = Value
      if autoCoinEnabled then
         notify("Auto Cày Xu", "Trạng thái: BẬT (Dịch chuyển)")
         
         task.spawn(function()
            local targetCFrame = CFrame.new(0.73, 5004.52, -84.40)
            
            while autoCoinEnabled do
               local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
               local hrp = character:WaitForChild("HumanoidRootPart", 5)
               
               if hrp then
                  hrp.CFrame = targetCFrame
                  
                  local raceEnded = false
                  local eventConn
                  
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
                  
                  if autoCoinEnabled then
                     task.wait(10)
                  end
               else
                  task.wait(1)
               end
            end
         end)
      else
         notify("Auto Cày Xu", "Trạng thái: TẮT")
      end
   end,
})

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
               Event:WaitForChild("Offline"):WaitForChild("ClaimOfflineReward"):FireServer()
               task.wait(10)
            end
         end)
      else
         notify("Tự nhận chất béo", "Trạng thái: TẮT")
      end
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
               Event:WaitForChild("Rebirth"):WaitForChild("TryRebirth"):FireServer()
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
      Event:WaitForChild("Rebirth"):WaitForChild("TryRebirth"):FireServer()
      notify("Tái sinh", "Đã thực hiện tái sinh ngay!")
   end,
})

FarmTab:CreateButton({
   Name = "Quay vòng quay",
   Callback = function()
      Function:WaitForChild("Spin"):WaitForChild("[C-S]TrySpin"):InvokeServer()
      notify("Vòng quay", "Đã thực hiện quay vòng quay!")
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
               Function:WaitForChild("Spin"):WaitForChild("[C-S]TrySpin"):InvokeServer()
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
      Event:WaitForChild("Pet"):WaitForChild("UnEquipAll"):FireServer()
      notify("Pet", "Đã gỡ tất cả pet đang trang bị!")
   end,
})

PetTab:CreateButton({
   Name = "Random trứng 1",
   Callback = function()
      local args = { "Egg1", 1 }
      Function:WaitForChild("Luck"):WaitForChild("[C-S]DoLuck"):InvokeServer(unpack(args))
      notify("Pet", "Đã thực hiện mở Trứng 1!")
   end,
})

PetTab:CreateButton({
   Name = "Random pet 2",
   Callback = function()
      local args = { "Egg2", 1 }
      Function:WaitForChild("Luck"):WaitForChild("[C-S]DoLuck"):InvokeServer(unpack(args))
      notify("Pet", "Đã thực hiện mở Trứng 2!")
   end,
})

PetTab:CreateButton({
   Name = "Random pet 3",
   Callback = function()
      local args = { "Egg3", 1 }
      Function:WaitForChild("Luck"):WaitForChild("[C-S]DoLuck"):InvokeServer(unpack(args))
      notify("Pet", "Đã thực hiện mở Trứng 3!")
   end,
})

PetTab:CreateButton({
   Name = "Random pet event free",
   Callback = function()
      Event:WaitForChild("PetEvent"):WaitForChild("TryOpenEventEgg"):FireServer()
      notify("Pet", "Đã gửi yêu cầu mở Pet Event Free!")
   end,
})

---------------------------------------------------------
-- TAB MUA ĐỒ
---------------------------------------------------------
local ShopTab = Window:CreateTab("Mua đồ", 4483362458)

ShopTab:CreateButton({
   Name = "Kiểm tra có bao nhiêu tiền",
   Callback = function()
      local moneyRaw = GetMoneyUniversal()
      if moneyRaw then
         local moneyNum = ParseMoney(moneyRaw)
         notify("Số tiền hiện tại", "Bạn đang có: " .. tostring(moneyRaw) .. " (" .. tostring(moneyNum) .. ")")
      else
         notify("Số tiền hiện tại", "Không tìm thấy dữ liệu số tiền!")
      end
   end,
})

-- TÍNH NĂNG MỚI: MUA THỨC ĂN TIẾP THEO
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
               local moneyRaw = GetMoneyUniversal()
               
               if moneyRaw then
                  local money = ParseMoney(moneyRaw)
                  
                  if money > 28500000 then
                     if tick() - lastNotifyTime > 15 then
                        notify("Thông báo", "Chưa được cập nhật hoặc cần qua thế giới 2")
                        lastNotifyTime = tick()
                     end
                  else
                     -- 1. Tìm thức ăn tương ứng với số tiền
                     local targetIndex = nil
                     for i, food in ipairs(foodList) do
                        if money >= food.min and money < food.max then
                           targetIndex = i
                           break
                        end
                     end
                     
                     -- 2. Thực hiện thử mua từ món cao nhất có thể về thấp hơn
                     if targetIndex then
                        for i = targetIndex, 1, -1 do
                           if not autoBuyFoodEnabled then break end
                           
                           local foodToBuy = foodList[i]
                           TryUnlockFood:FireServer(foodToBuy.name)
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
               
               task.wait(3) -- Kiểm tra lại sau mỗi 3 giây
            end
         end)
      else
         notify("Auto Thức Ăn", "Trạng thái: TẮT")
      end
   end,
})

ShopTab:CreateButton({
   Name = "Mua đường mòn nước",
   Callback = function()
      local args = { "Blister" }
      Event:WaitForChild("Trail"):WaitForChild("TryUnlockTrail"):FireServer(unpack(args))
      notify("Cửa hàng", "Đã gửi yêu cầu mua Đường mòn nước!")
   end,
})

ShopTab:CreateButton({
   Name = "Mua đường mòn sấm sét",
   Callback = function()
      local args = { "Lightning" }
      Event:WaitForChild("Trail"):WaitForChild("TryUnlockTrail"):FireServer(unpack(args))
      notify("Cửa hàng", "Đã gửi yêu cầu mua Đường mòn sấm sét!")
   end,
})

ShopTab:CreateButton({
   Name = "Mua đường mòn nhạc",
   Callback = function()
      local args = { "Music" }
      Event:WaitForChild("Trail"):WaitForChild("TryUnlockTrail"):FireServer(unpack(args))
      notify("Cửa hàng", "Đã gửi yêu cầu mua Đường mòn nhạc!")
   end,
})

---------------------------------------------------------
-- TAB THÔNG TIN NGƯỜI CHƠI
---------------------------------------------------------
local PlayerTab = Window:CreateTab("Thông Tin Người Chơi", 4483362458)

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
               if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                  LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = speedValue
               end
               task.wait(0.1)
            end
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
               LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
            end
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
               if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
                  local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                  hum.UseJumpPower = true
                  hum.JumpPower = jumpValue
               end
               task.wait(0.1)
            end
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
               LocalPlayer.Character:FindFirstChildOfClass("Humanoid").JumpPower = 50
            end
         end)
      else
         notify("Nhảy cao", "Đã TẮT độ cao nhảy (Trở về mặc định)")
      end
   end,
})

-- Tự động khôi phục cấu hình đã lưu
Rayfield:LoadConfiguration()
