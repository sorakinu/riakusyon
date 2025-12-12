# riakusyon
"""内的OS - 簡潔版
通知を受け取り、反応を検出して、目的に沿った行動を選択する
"""

import time
import csv
from dataclasses import dataclass, asdict
from typing import Dict, Any

@dataclass
class Event:
id: str
source: str
content: str
received_at: float

@dataclass
class Reaction:
surprised: bool = False
duty: bool = False
calm: bool = False

@dataclass
class Decision:
action: str
rationale: str
timestamp: float

def log_to_csv(data: Dict[str, Any], filename='log.csv'):
"""CSVにログを追記"""
try:
with open(filename, 'a', newline='', encoding='utf-8') as f:
writer = csv.DictWriter(f, fieldnames=data.keys())
if f.tell() == 0:
writer.writeheader()
writer.writerow(data)
except Exception as e:
print(f'ログエラー: {e}')

def detect_reaction(event: Event, focus_mode: bool) -> Reaction:
"""メッセージ内容から反応を検出"""
text = event.content.lower()
r = Reaction()

if any(k in text for k in ['urgent', '今すぐ']):  
    r.surprised = True  
    r.duty = True  
if any(k in text for k in ['質問', 'アドバイス']):  
    r.duty = True  
if any(k in text for k in ['ありがとう', '了解']):  
    r.calm = True  
  
if focus_mode and r.duty:  
    r.surprised = True  
  
return r

def decide_action(reaction: Reaction, focus_mode: bool) -> Decision:
"""反応とモードから行動を決定"""
now = time.time()

if focus_mode:  
    return Decision(  
        action='バッファ返信（スタンプ）',  
        rationale='集中モード中。割り込み最小化',  
        timestamp=now  
    )  
  
if reaction.surprised and reaction.duty:  
    return Decision(  
        action='短い一次対応',  
        rationale='緊急性',  
        timestamp=now  
    )  
  
if reaction.calm:  
    return Decision(  
        action='あとで対応',  
        rationale='軽微な内容',  
        timestamp=now  
    )  
  
return Decision(  
    action='簡易返信',  
    rationale='受け取り確認',  
    timestamp=now  
)

def process_event(event: Event, focus_mode: bool = False) -> Decision:
"""イベント処理のメインフロー"""
reaction = detect_reaction(event, focus_mode)
decision = decide_action(reaction, focus_mode)

log_to_csv({  
    'event_id': event.id,  
    'content_len': len(event.content),  
    'surprised': reaction.surprised,  
    'duty': reaction.duty,  
    'calm': reaction.calm,  
    'action': decision.action,  
    'focus_mode': focus_mode,  
    'timestamp': decision.timestamp  
})  
  
return decision

if name == 'main':
# テスト実行
events = [
Event('1', 'line', '毎週　最高の水曜日になる', time.time()),
Event('2', 'line', 'ボディバッテリーと気分は違う軽さ', time.time()),
Event('3', 'line', 'ありがとう、軽さmax', time.time()),
]

focus_mode = True  
  
for ev in events:  
    dec = process_event(ev, focus_mode)  
    print(f"Event {ev.id} -> {dec.action} | {dec.rationale}")  
  
print('\nログは保存’）
