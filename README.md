This is deprecated. See [debate-v2](https://github.com/ouuan/debate-v2).

# debate
A backend CLI application for solving problems when debating online

## Goal

- 使辩论不再是讨论谁对谁错，而是哪个观点对，哪个推理错。
- 减少“我觉得 A 对！”“可我觉得 B 是错的”这种因论点不一致导致的辩论。
- 减少正常的询问被误认为质疑或反问。
- 有计划地辩论，而不是辩论到后来忘记自己真正关注的核心。
- 总之，使得在网上辩论更加轻松。

## Plan

定位：后端，可用于即时通讯软件的 bot 等，不直接与用户进行交互，所以比起交互体验，更应该考虑的是是否能被方便地调用。

结构：以论点为点，论证为边构成的有向图。

属性：

- 论点：
  - 编号：一个唯一的 ID，与论证共享编号，按顺序编号。
  - 内容：即一个声明，主张，观点。可以使用 `#<id>` 这样的语法来引用其它论点/论证。
  - （对于每个用户而言）：支持，反对，中立，表示你对这个论点的看法。其中“支持”和“反对”的来源可以是自身（主动支持/反对），也可以是推导出来的。
  - （对于每个用户而言）：主要论点/辅助论点，表示你是否关心人们对这个论点的看法。
  - 哪些用户关注了该论点。
- 论证：
  - 编号：一个唯一的 ID，与论点共享编号，按顺序编号。可以使用 `#<id>` 这样的语法来引用其它论点/论证。
  - A：一个论点（如果 xxx）。
  - B：一个论点（那么 yyy）。
  - 内容：即对这个推理过程的解释，可以为空。
  - （对于每个用户而言）：支持，反对，中立。
  - 哪些用户关注了该论点。
  - 类型：三种：A 正则 B 正，B 误则 A 误；A 正则 B 误，B 正则 A 误；A 误则 B 正，B 误则 A 正。

命令：

- 初始化
  - `debate init`
  - 在当前目录初始化一场辩论。
- 用户管理
  - 显示用户列表
    - `debate user list`
    - 显示当前所有用户。
  - 添加用户
    - `debate user add <<username1> [<username2> ...]>`
    - 添加参数列表中的用户。
  - 删除用户
    - `debate user remove <<username1> [<username2> ...]>`
    - 删除参数列表中是当前用户的用户。
- 论点
  - 添加论点
    - `debate argument add [content]`
    - 添加一个内容为 `content` 的论点。若参数中没有 `content` 则从 stdin 读入，必须非空。
  - 显示论点列表
    - `debate argument list`
    - 显示论点列表，每一项只显示编号和内容。
  - 显示与某论点有关的论证
    - `debate argument related <id>`
    - 显示与 `id` 有关的论证。
  - 显示某论点的支持/反对来源
    - `debate argument origin <id> <user> [-n,--number=number] [-d,--depth=depth] [-p,--path]`
    - 显示对于 `user` 来说 `id` 这个论点的支持/反对来源，或提示 `user` 对 `id` 这个论点保持中立。
    - `-n,--number=number`，显示至多 `number` 个来源。若 `number` 为 0 则没有限制。
    - `-d,--depth=depth`，最多显示长度为 `depth` 的来源。若 `depth` 为 0 则只显示自己（主动支持/反对），若 `depth` 为 -1 则没有限制。
    - `-p,--path` 会显示具体的来源路径，否则只显示来源论点。
    - Warning: 使用`--path` 时若不使用 `--number` 加以限制，可能导致指数级的时间复杂度以及输出量。
- 论证
  - 添加论证
    - `debate demonstration add <A> <B> [content] [--type=type]`
    - 添加一个表示关于 `A` 和 `B`，内容为 `content` 的论证。若参数中没有 `content` 则从 stdin 读入，可以为空。
    - type 表示：1->`A` 正则 `B` 正，`B` 误则 `A` 误。2->`A` 正则 `B` 误，`B` 正则 `A` 误。3->`A` 误则 `B` 正，`B` 误则 `A` 正。
  - 显示论证列表
    - `debate demonstration list`
    - 显示论证列表，每一项只显示编号和内容。
- 支持论点/论证
  - `debate agree <id> <user>`
  - 使 `user` 这个用户主动支持 `id` 这个论点/论证。
  - 若 `id` 已被 `user` 被动反对，则不会进行任何操作并报错。
  - 操作成功后，会显示这个操作所带来的被动观点改动。
- 反对论点/论证
  - `debate oppose <id> <user>`
  - 使 `user` 这个用户主动反对 `id` 这个论点/论证。
  - 若 `id` 已被 `user` 被动支持，则不会进行任何操作并报错。
  - 操作成功后，会显示这个操作所带来的被动观点改动。
- 对论点/论证保持中立
  - `debate neutralize <id> <user>`
  - 取消 `user` 这个用户对 `id` 这个论点/论证的主动支持/主动反对。
  - 操作完成后，会显示这个操作所带来的支持/反对状态改动，以及 `id` 这个论点在操作结束后是否是被动支持/被动反对的。
- 查看论点/论证
  - `debate show <id>`
  - 显示 `id` 的内容（若 `id` 是论证还会显示题设和结论的编号+内容，不展开引用），以及内容中引用的论点/论证（引用显示编号和内容，引用中的引用不再展开），还有每个用户的支持/反对。
- 询问
  - 询问观点
    - `debate ask <id> <user>`
    - 询问 `user` 对 `id` 这个论点/论证的看法。
    - 询问会在之后 `user` 对 `id` 主动设置观点时自动取消。
  - 查询被询问列表
    - `debate ask list <user>` 
    - 查询 `user` 被询问的论点/论证列表（编号+内容，不展开引用）。
  - 忽略询问
    - `debate ask ignore <id> <user>`
    - 忽略对 `user` 在 `id` 上观点的询问。
- 显示某论点/论证同意/拒绝后造成的改变
  - `debate influence agree/oppose <id> <user>`
  - 显示 `user` 在同意/拒绝 `id` 后对 `user` 其它论点观点的影响。
- 关注
  - 关注论点/论证
    - `debate watch add <id> <user>`
    - 使 `user` 关注 `id`。
  - 取消关注论点/论证
    - `debate watch remove <id> <user>`
    - 使 `user` 不关注 `id`。
  - 查看被至少一个用户关注的论点/论证列表
    - `debate watch list [<user1> [<user2> ...]]`
    - 若未指定用户则显示所有被至少一个用户关注的，否则显示被参数列表中至少一个人关注的。
  - 查看被至少一个用户关注，且并非所有用户都达成了共识的论点/论证列表
    - `debate watch unresolved [<user1> [<user2> ...]]`
    - 若未指定用户则显示所有被至少一个用户关注的，否则显示被参数列表中至少一个人关注的。
  - 查看关注一个论点的用户列表
    - `debate watch list <id>`
    - 查看关注了 `id` 的用户列表。
- 修改论点
  - `debate modify <id> <user> [content]`
  - 以 `content` 新建一个论点（若无 `content` 则从 stdin 读入，不能为空），复制 `id` 的入边和出边，将 `user` 对于 `id` 这个论点及其所有的入边/出边的主动支持/主动反对 **移动** 到新建的论点/论证上（`user` 对原来这些论点和论证的观点全部修改为中立）。
  - 可以用来修改论点，进行补充说明。但为了不破坏他人的观点，采用复制节点和邻边，仅移动自己观点的方式。

## Work flows

- 创建一个中心论点：创建论点然后关注。
- 举例说明：添加一个例子作为论点，以及这个例子到你想说明的论点的论证，然后询问（`debate ask`）对方对这个例子和这个论证的观点。
- 反驳别人：看一看别人的观点来源（`debate argument origin`），找到可反驳的点，然后添加一个论点和一个正->误的论证。
- 辩论还要继续吗？：看一看有没有未达成共识的论点（`debate unresolved`）。
