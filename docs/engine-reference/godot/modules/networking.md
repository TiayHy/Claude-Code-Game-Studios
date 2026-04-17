# Godot 网络 —— 速查表

最后验证：2026-02-12 | 引擎：Godot 4.6

## 自 ~4.3（LLM 截止日期）以来的变化

### 4.6 变化
- **网络部分的破坏性变更**：4.5→4.6 级别的具体内容见官方迁移指南

### 4.5 变化
- **无重大网络 API 破坏** —— 核心多人 API 保持稳定

## 当前 API 模式

### 高层多人游戏
```gdscript
# 服务器
func host_game(port: int = 9999) -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_server(port)
    multiplayer.multiplayer_peer = peer
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)

# 客户端
func join_game(address: String, port: int = 9999) -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_client(address, port)
    multiplayer.multiplayer_peer = peer
```

### RPC
```gdscript
# 服务器权威模式
@rpc("any_peer", "call_local", "reliable")
func request_action(action_data: Dictionary) -> void:
    if not multiplayer.is_server():
        return
    # 在服务器验证后再广播
    _execute_action.rpc(action_data)

@rpc("authority", "call_local", "reliable")
func _execute_action(action_data: Dictionary) -> void:
    # 所有端执行已验证的动作
    pass
```

### MultiplayerSpawner 和 MultiplayerSynchronizer
```gdscript
# 使用 MultiplayerSpawner 自动复制节点
# 使用 MultiplayerSynchronizer 同步属性

# MultiplayerSynchronizer 配置：
# 1. 添加为要同步的节点的子节点
# 2. 在编辑器中配置要复制的属性
# 3. 设置可见性过滤器以控制相关性
```

### SceneMultiplayer 配置
```gdscript
func _ready() -> void:
    var scene_mp := multiplayer as SceneMultiplayer
    scene_mp.auth_callback = _authenticate_peer
    scene_mp.server_relay = false  # 直接端到端连接

func _authenticate_peer(id: int, data: PackedByteArray) -> void:
    # 自定义认证逻辑
    pass
```

## 常见错误
- 客户端到服务器的 RPC 未使用 `"any_peer"`（默认仅为权威）
- 客户端数据未经服务器端验证就信任
- 对游戏状态变更使用 `"unreliable"`（仅用于位置更新）
- 未在生成节点上设置多人权威（`set_multiplayer_authority()`）