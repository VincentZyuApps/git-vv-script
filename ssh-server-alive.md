## SSH Server Alive

### Windows Powershell
```
nano $PROFILE
```
然后，以下的文本放到 `$PROFILE`
```powershell
# ----- ↓ sshsa helper start 🚀 -----
function sshsa {
    $UseIpv4 = $false
    $PositionalArgs = @()

    foreach ($Arg in $args) {
        if ($Arg -eq "--ipv4") {
            $UseIpv4 = $true
        }
        else {
            $PositionalArgs += $Arg
        }
    }

    $Target = if ($PositionalArgs.Count -gt 0) { [string]$PositionalArgs[0] } else { $null }
    $Port = if ($PositionalArgs.Count -gt 1) { [int]$PositionalArgs[1] } else { 22 }

    # 检查是否请求帮助
    if ($Target -eq "-h" -or $Target -eq "--help" -or [string]::IsNullOrEmpty($Target)) {
        Write-Host "`nUsage: sshsa [--ipv4] <user@host> [port]" -ForegroundColor Yellow
        Write-Host "Example: sshsa root@1.1.1.1 2222"
        Write-Host "Example: sshsa --ipv4 root@1.1.1.1 2222"
        Write-Host "Options: Defaults to port 22. Use --ipv4 to force IPv4. Includes 60s keep-alive.`n"
        return
    }

    # 执行 SSH
    $SshArgs = @()
    if ($UseIpv4) {
        $SshArgs += "-4"
    }
    $SshArgs += @("-p", $Port, "-o", "ServerAliveInterval=60", "-o", "ServerAliveCountMax=3", $Target)
    ssh @SshArgs
}
# ----- ↑ sshsa helper end ✅ -----
```
然后：
```powershell
. $PROFILE
sshsa --help
```

### Linux Bash
```bash
nano ~/.bashrc
```
然后，以下的文本放到 `~/.bashrc`
```bash
# ----- ↓ sshsa helper start 🚀 -----
sshsa() {
    local USE_IPV4=0
    local POSITIONAL=()

    while [[ $# -gt 0 ]]; do
        case "$1" in
            --ipv4)
                USE_IPV4=1
                shift
                ;;
            -h|--help)
                echo -e "\nUsage:  sshsa [--ipv4] <user@host> [port]"
                echo -e "Example: sshsa root@1.1.1.1 2222"
                echo -e "Example: sshsa --ipv4 root@1.1.1.1 2222"
                echo -e "Note:    Defaults to port 22. Use --ipv4 to force IPv4. Sends heartbeats every 60s.\n"
                return 0
                ;;
            *)
                POSITIONAL+=("$1")
                shift
                ;;
        esac
    done

    local TARGET=${POSITIONAL[0]}
    local PORT=${POSITIONAL[1]:-22}

    # 检查空参数
    if [[ -z "$TARGET" ]]; then
        echo -e "\nUsage:  sshsa [--ipv4] <user@host> [port]"
        echo -e "Example: sshsa root@1.1.1.1 2222"
        echo -e "Example: sshsa --ipv4 root@1.1.1.1 2222"
        echo -e "Note:    Defaults to port 22. Use --ipv4 to force IPv4. Sends heartbeats every 60s.\n"
        return 0
    fi

    local SSH_ARGS=()
    if [[ "$USE_IPV4" -eq 1 ]]; then
        SSH_ARGS+=("-4")
    fi
    SSH_ARGS+=("-p" "$PORT" "-o" "ServerAliveInterval=60" "-o" "ServerAliveCountMax=3" "$TARGET")

    if [[ "$USE_IPV4" -eq 1 ]]; then
        echo "Connecting to $TARGET on port $PORT with IPv4 keep-alive..."
    else
        echo "Connecting to $TARGET on port $PORT with keep-alive..."
    fi
    ssh "${SSH_ARGS[@]}"
}
# ----- ↑ sshsa helper end ✅ -----
```
然后：
```bash
. ~/.bashrc
sshsa --help
```
