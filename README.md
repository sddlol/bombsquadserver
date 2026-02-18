**skid by qinuan bombsquad sever

>直接把bss.zip下载解压就行


##基本命令介绍
---

加管理
-

```bash
import sys
sys.path.append('data/scripts')
import membersID as MID

pid = 'pb-xxxxxxxcxxx'

# 加入 admins
MID.persist('admins', list(MID.admins) + [pid])
# 从普通人列表移除
MID.persist('members_count', [x for x in MID.members_count if str(x) != pid])

print('OK promote to admin:', pid)
```
禁人
-
```bash
# -*- coding: utf-8 -*-
import sys
sys.path.append('data/scripts')

import membersID as MID
try:
    import bsInternal
except Exception:
    bsInternal = None

# ====== 改这里：/list 里的 clientID ======
CID = 1
# ======================================

pbid = None
name = ''

# 先拿显示名（如果能拿到）
if bsInternal is not None:
    try:
        for c in bsInternal._getGameRoster():
            try:
                if c.get('clientID') == CID:
                    name = c.get('displayString', '') or ''
                    break
            except Exception:
                pass
    except Exception:
        pass

# 再拿 pb-id
if bsInternal is not None:
    try:
        act = bsInternal._getForegroundHostActivity()
        if act is not None:
            for p in act.players:
                try:
                    if p.getInputDevice().getClientID() == CID:
                        pbid = p.get_account_id()
                        break
                except Exception:
                    pass
    except Exception:
        pass

if not pbid:
    print('FAIL: 找不到该 clientID 的 pb-id, CID=', CID)
else:
    pbid = str(pbid)

    # 1) 加入 rejected 黑名单（永久）
    rejected = [str(x) for x in getattr(MID, 'rejected', [])]
    if pbid not in rejected:
        rejected.append(pbid)
    MID.persist('rejected', rejected)

    # 2) 可选：从 members_count(普通人列表) 移除，避免你说的“冲突”
    try:
        if hasattr(MID, 'members_count'):
            MID.persist('members_count', [x for x in MID.members_count if str(x) != pbid])
    except Exception:
        pass

    print('BANNED OK:', pbid, 'CID=', CID, 'NAME=', name)

    # 3) 尝试立刻踢掉（不同服接口可能不同，失败也不会炸）
    kicked = False
    if bsInternal is not None:
        # 常见的一些断开接口名，能用哪个算哪个
        for fn in ['_disconnectClient', 'disconnectClient', '_kickClient', 'kickClient']:
            try:
                f = getattr(bsInternal, fn, None)
                if f is not None:
                    f(CID)
                    kicked = True
                    break
            except Exception:
                pass

    print('KICK TRY:', 'OK' if kicked else 'NO-API/FAIL (下次进服会被拒绝)')
```
