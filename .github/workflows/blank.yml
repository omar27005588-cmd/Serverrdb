# ---
# Workflow_Dispatch
# ---

# Name: RDP-Windows-Setup
# On: workflow_dispatch

# Jobs:
#   RDP-Setup:
#     Runs-on: Windows-Latest
#     Steps:
#       - Name: Configure Core RDP Settings
#         Run: |
#           # Enable Remote Desktop and disable Network Level Authentication (if needed)
#           Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
#           Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name "UserAuthentication" -Value 0
#           Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name "SecurityLayer" -Value 1
#           
#           # Ensure any existing rule with the same name is removed first
#           Remove-NetFirewallRule -DisplayName "Allow RDP" -ErrorAction SilentlyContinue
#           
#           # Explicitly allow any incoming traffic on port 3389
#           New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -LocalPort 3389 -Protocol TCP -Action Allow
#           New-NetFirewallRule -DisplayName "Allow RDP UDP" -Direction Inbound -LocalPort 3389 -Protocol UDP -Action Allow
#           
#           # (Optional) Restart the Remote Desktop Service to ensure changes take effect
#           Restart-Service -Name "TermService" -Force

#       - Name: Create RDP User with Secure Password
#         Run: |
#           $Uppercase = @(65..90) | ForEach-Object {[char]$_}
#           $Lowercase = @(97..122) | ForEach-Object {[char]$_}
#           $Digits = @(48..57) | ForEach-Object {[char]$_}
#           $Special = @(33,35,36,37,64) | ForEach-Object {[char]$_} # Selected special characters
#           
#           $GenPassword = {
#               $charList = $Uppercase * 3 + $Lowercase * 3 + $Digits * 3 + $Special * 3
#               $password = (Get-Random -InputObject $charList -Count 15) -join ""
#               return $password
#           }
#           
#           $securePassword = &$GenPassword
#           $secureStringPassword = ConvertTo-SecureString $securePassword -AsPlainText -Force
#           New-LocalUser -Name "Administrator" -Password $secureStringPassword -Description "Automated RDP User"
#           Add-LocalGroupMember -Group "Administrators" -Member "Administrator"
#           Add-LocalGroupMember -Group "Remote Desktop Users" -Member "Administrator"
#           
#           Write-Host "RDP User Created. Username: Administrator, Password: $securePassword"

#       # Find out Tailscale Auth Key
#       # If Final-Auth-Key: "tskey-auth-..."
#       # If Not: "tskey-auth-..."

#       - Name: Install Tailscale
#         Run: |
#           Invoke-WebRequest "https://tailscale.com" -OutFile "tailscale-setup.msi"
#           Start-Process msiexec.exe -ArgumentList "/i tailscale-setup.msi /qn /norestart" -Wait
#           Remove-Item tailscale-setup.msi
#           
#           # Ensure path environment variable includes Tailscale
#           $env:Path += ";C:\Program Files\Tailscale"

#       - Name: Authenticate Tailscale
#         Run: |
#           # Bring up Tailscale with the provided auth key and set a unique hostname
#           & "C:\Program Files\Tailscale\tailscale.exe" up --authkey=${{ secrets.TAILSCALE_AUTH_KEY }} --accept-routes
#           
#           # Wait for Tailscale to assign an IP
#           $assignedIP = $null
#           $retries = 0
#           while ($null -eq $assignedIP -and $retries -lt 10) {
#               Start-Sleep -Seconds 5
#               $ipOutput = & "C:\Program Files\Tailscale\tailscale.exe" ip -4
#               if ($ipOutput -match '^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$') {
#                   $assignedIP = $ipOutput
#               }
#               $retries++
#           }
#           
#           if ($null -eq $assignedIP) {
#               Write-Error "Timeout: Tailscale IP not assigned. Exiting."
#               exit 1
#           }
#           
#           Write-Host "Tailscale IP Assigned: $assignedIP"

#       - Name: Verify RDP Accessibility
#         Run: |
#           # Fetch Tailscale IP
#           $tsIP = & "C:\Program Files\Tailscale\tailscale.exe" ip -4
#           
#           # Test internal TCP connectivity from local host against the Tailscale IP on port 3389
#           $tcpResult = Test-NetConnection -ComputerName $tsIP -Port 3389
#           if (-not $tcpResult.TcpTestSucceeded) {
#               Write-Error "RDP service is not listening on the Tailscale IP."
#               exit 1
#           }
#           
#           Write-Host "RDP Connectivity successful."

#       - Name: Maintain Connection
#         Run: |
#           # Output login info
#           $tsIP = & "C:\Program Files\Tailscale\tailscale.exe" ip -4
#           Write-Host "Address: $tsIP:3389"
#           Write-Host "Username: Administrator"
#           Write-Host "Password: Check RDP User Creation job output"
#           
#           # Keep tunnel active indefinitely (or until manually cancelled)
#           while ($true) {
#               Write-Host "Tunnel status: Active. Use Ctrl+C in workflow to terminate."
#               Start-Sleep -Seconds 300
#           }
