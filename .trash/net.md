[Skip to main content  
跳至主要内容](https://cedric-neukirchen.net/docs/multiplayer-compendium/network-in-unreal#docusaurus_skipToContent_fallback)

[

![Unreal Engine Logo](https://cedric-neukirchen.net/images/logo.svg)

**Blog Posts**](https://cedric-neukirchen.net/)

[

![Unreal Engine Logo](https://cedric-neukirchen.net/images/logo.svg)

](https://cedric-neukirchen.net/)

- [](https://cedric-neukirchen.net/docs/intro)
- [](https://saltypandastudios.com/)
- [](https://twitter.com/exifrexi)
- [](https://github.com/exifrexi)

- [](https://cedric-neukirchen.net/docs/intro)
- [](https://cedric-neukirchen.net/docs/category/multiplayer-network-compendium)
    
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/introduction)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/network-in-unreal)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/)
        
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/framework-and-network)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/dedicated-vs-listen-server)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/replication)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/remote-procedure-calls)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/ownership)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/actor-relevancy-and-priority)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/actor-roles)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/traveling-in-multiplayer)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/start-multiplayer-game)
    - [](https://cedric-neukirchen.net/docs/multiplayer-compendium/additional-resources)
- [](https://cedric-neukirchen.net/docs/category/session-management)
    

- [](https://cedric-neukirchen.net/)
- [Multiplayer Network Compendium](https://cedric-neukirchen.net/docs/category/multiplayer-network-compendium)
- Network in Unreal

On this page

# Network in Unreal 虚幻中的网络

Unreal Engine uses a standard **Server-Client** architecture. This means the server is **authoritative** and all data must be sent from the client to the server first. After which the server validates the data and reacts depending on your code.  
虚幻引擎使用标准的服务器-客户端架构。这意味着服务器是权威的，所有数据必须首先从客户端发送到服务器。之后，服务器验证数据并根据您的代码做出反应。

## A Small Example[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/network-in-unreal#a-small-example "Direct link to A Small Example") 一个小例子​

When you move your character as a client in a multiplayer match, you don't move your character by yourself but tell the server that you want to move it. The server then updates the transform of the character for everyone else, including you.  
当您作为客户端在多人游戏中移动角色时，您不会自己移动角色，而是告诉服务器您想要移动它。然后，服务器会为其他人（包括您）更新角色的变换。

INFO 信息

To prevent a feeling of “lag” for the local client, programmers often, in addition, let the local client directly control their character -- although the server still might override the character's Location when the client starts cheating! This means the client will (almost) never 'talk' to other clients directly.  
此外，为了防止本地客户端有“滞后”的感觉，程序员通常还让本地客户端直接控制他们的角色——尽管当客户端开始作弊时，服务器仍然可能覆盖角色的位置！这意味着客户（几乎）永远不会直接与其他客户“交谈”。

## Another Example[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/network-in-unreal#another-example "Direct link to Another Example") 另一个例子​

When sending a chat message to another client you are sending it to the server first, which then passes it to the client you wanted to reach. This could also be a team, guild, group, etc.  
当向另一个客户端发送聊天消息时，您首先将其发送到服务器，然后服务器将其传递给您想要联系的客户端。这也可以是团队、公会、团体等。

IMPORTANT 重要的

**Never** trust the client! Trusting the client here means you don't test the client's actions before executing them.  
永远不要相信客户！这里信任客户端意味着您在执行客户端的操作之前不会测试它们。

**This will allow them to cheat!  
这会让他们作弊！**

A simple example would be firing a weapon: Make sure to test, on the server, if the client has the required amount of ammo and is allowed to shoot instead of directly processing the shot!  
一个简单的例子是发射武器：确保在服务器上测试客户端是否拥有所需数量的弹药并且允许射击而不是直接处理射击！

[

Previous 以前的

Introduction 介绍

](https://cedric-neukirchen.net/docs/multiplayer-compendium/introduction)[

Next 下一个

Gameplay Framework 游戏框架

](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/)

Copyright © 2023 Cedric Neukirchen. Built with Docusaurus.