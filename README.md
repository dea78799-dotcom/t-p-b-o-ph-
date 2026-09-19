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
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remote = ReplicatedStorage:WaitForChild("Remote")
local Event = Remote:WaitForChild("Event")
local Function = Remote:WaitForChild("Function")

local EatEvent = Event:WaitForChild("Eat")
local PlayerIsEat = EatEvent:WaitForChild("PlayerIsEat")
local PlayerTryClickRE = EatEvent:WaitForChild("PlayerTryClickRE")
local PlayerEndRace = Event:WaitForChild("Race"):WaitForChild("PlayerEndRace")

-- Variables
local autoTrainEnabled = false
local autoClaimOfflineEnabled = false
local autoRebirthEnabled = false
local autoSpinEnabled = false
local autoCoinEnabled = false

local speedEnabled = false
local jumpEnabled = false
local speedValue = 16
local jumpValue = 50

-- Hàm hiển thị thông báo
local function notify(title, content)
   Rayfield:Notify({
      Title = title,
      Content = content,
      Duration = 3,
      Image = 4483362458,
   })
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
         notify("Auto Cày Xu", "Trạng thái: BẬT")
         
         task.spawn(function()
            local targetCFrame = CFrame.new(0.73, 5004.52, -84.40)
            
            while autoCoinEnabled do
               local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
               local hrp = character:WaitForChild("HumanoidRootPart", 5)
               
               if hrp then
                  -- Tính thời gian bay dựa trên khoảng cách (Tốc độ bay mượt)
                  local distance = (hrp.Position - targetCFrame.Position).Magnitude
                  local tweenInfo = TweenInfo.new(distance / 100, Enum.EasingStyle.Linear)
                  local tween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
                  
                  tween:Play()
                  
                  -- Chờ đến khi bay xong hoặc dừng nếu tắt toggle
                  local completed = false
                  local conn
                  conn = tween.Completed:Connect(function()
                     completed = true
                     if conn then conn:Disconnect() end
                  end)
                  
                  while not completed and autoCoinEnabled do
                     task.wait(0.1)
                  end
                  
                  if not autoCoinEnabled then
                     tween:Cancel()
                     break
                  end
                  
                  -- Chờ sự kiện PlayerEndRace kích hoạt từ server
                  local raceEnded = false
                  local eventConn
                  eventConn = PlayerEndRace.OnClientEvent:Connect(function()
                     raceEnded = true
                     if eventConn then eventConn:Disconnect() end
                  end)
                  
                  -- Đợi tối đa hoặc chờ tín hiệu từ server
                  local timeout = 0
                  while not raceEnded and autoCoinEnabled and timeout < 30 do
                     task.wait(0.5)
                     timeout = timeout + 0.5
                  end
                  
                  if eventConn then eventConn:Disconnect() end
                  
                  -- Chờ 5 giây trước khi lặp lại vòng mới
                  if autoCoinEnabled then
                     task.wait(5)
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

---------------------------------------------------------
-- TAB MUA ĐỒ
---------------------------------------------------------
local ShopTab = Window:CreateTab("Mua đồ", 4483362458)

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
