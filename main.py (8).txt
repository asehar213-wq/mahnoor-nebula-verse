
from kivy.app import App
from kivy.core.window import Window
from kivy.core.audio import SoundLoader
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.screenmanager import ScreenManager, Screen
from kivy.uix.scrollview import ScrollView
from kivy.uix.gridlayout import GridLayout
from kivy.uix.popup import Popup
from kivy.graphics import Color, Rectangle, Line
from kivy.uix.floatlayout import FloatLayout
import random, os, wave, struct, math

Window.clearcolor = (0.95,0.85,0.95,1)

# ===== AUTO CREATE SOUND FILES - NO NEED TO DOWNLOAD =====
def create_sounds():
    try:
        os.makedirs("sounds", exist_ok=True)
        def tone(path, freq, dur, vol=0.4):
            if os.path.exists(path): return
            fr=44100; nf=int(fr*dur)
            with wave.open(path,'w') as w:
                w.setnchannels(1); w.setsampwidth(2); w.setframerate(fr)
                for i in range(nf):
                    t=i/fr
                    s=int(vol*32767*math.sin(2*math.pi*freq*t)*math.exp(-t*1.5))
                    w.writeframes(struct.pack('<h', s))
        def bgm(path):
            if os.path.exists(path): return
            fr=44100; dur=3.0; freqs=[261,329,392,523]
            with wave.open(path,'w') as w:
                w.setnchannels(1); w.setsampwidth(2); w.setframerate(fr)
                nf=int(fr*dur)
                for i in range(nf):
                    t=i/fr; idx=int(t*2)%len(freqs); f=freqs[idx]
                    s=int(0.25*32767*math.sin(2*math.pi*f*t))
                    w.writeframes(struct.pack('<h', s))
        tone("sounds/win.wav", 880, 0.5, 0.5)
        tone("sounds/lose.wav", 220, 0.6, 0.4)
        tone("sounds/click.wav", 700, 0.15, 0.3)
        bgm("sounds/bgm.wav")
        print("Sounds Created!")
    except Exception as e:
        print("Sound Error", e)

create_sounds()

class SMusic:
    def __init__(self):
        self.bgm=None; self.win=None; self.lose=None; self.click=None
        try:
            if os.path.exists("sounds/bgm.wav"):
                self.bgm=SoundLoader.load("sounds/bgm.wav")
                if self.bgm: self.bgm.loop=True
            if os.path.exists("sounds/win.wav"): self.win=SoundLoader.load("sounds/win.wav")
            if os.path.exists("sounds/lose.wav"): self.lose=SoundLoader.load("sounds/lose.wav")
            if os.path.exists("sounds/click.wav"): self.click=SoundLoader.load("sounds/click.wav")
        except: pass
    def play(self, name):
        try:
            if name=="win" and self.win: self.win.play()
            elif name=="lose" and self.lose: self.lose.play()
            elif name=="click" and self.click: self.click.play()
        except: pass
    def music_on(self):
        try:
            if self.bgm: self.bgm.play()
        except: pass
    def music_off(self):
        try:
            if self.bgm: self.bgm.stop()
        except: pass

SOUND = SMusic()

C = ["Pakistan","India","USA","UK","Turkey","Saudi","UAE","Canada","Japan","Korea","Germany","France","Brazil","Italy","Spain"]
L = ["English","Urdu","Hindi","Arabic","Turkish"]
T = {"English":{"w":"Welcome!","c":"Country","l":"Language","s":"START","lv":"LEVEL","tg":"TARGET","li":"LIVES","co":"COINS","ch":"Choose!","wi":"WIN!","lo":"LOSE!","nx":"NEXT","se":"SETTINGS","ba":"BACK","sh":"SHOP","ch2":"CHARS","ho":"HOME","ch3":"CHAT","ra":"RANK"},"Urdu":{"w":"Khush Amdeed!","c":"Mulk","l":"Zaban","s":"Shuru Karo","lv":"Level","tg":"Hadaf","li":"Jaan","co":"Sikke","ch":"Chuno!","wi":"Jeet!","lo":"Haar!","nx":"Agla Level","se":"Settings","ba":"Wapas","sh":"Dukan","ch2":"Kirdar","ho":"Ghar","ch3":"Chat","ra":"Rank"},"Hindi":{"w":"Swagat!","c":"Desh","l":"Bhasha","s":"Shuru","lv":"Star","tg":"Lakshya","li":"Jaan","co":"Sikke","ch":"Chuno!","wi":"Jeet!","lo":"Haar!","nx":"Agla","se":"Settings","ba":"Wapas","sh":"Dukan","ch2":"Patra","ho":"Ghar","ch3":"Chat","ra":"Rank"},"Arabic":{"w":"Ahlan!","c":"Country","l":"Lang","s":"START","lv":"Level","tg":"Target","li":"Lives","co":"Coins","ch":"Choose!","wi":"Win!","lo":"Lose!","nx":"Next","se":"Settings","ba":"Back","sh":"Shop","ch2":"Chars","ho":"Home","ch3":"Chat","ra":"Rank"},"Turkish":{"w":"Hosgeldin!","c":"Ulke","l":"Dil","s":"BASLA","lv":"Seviye","tg":"Hedef","li":"Can","co":"Coins","ch":"Sec!","wi":"Kazandin!","lo":"Kaybettin!","nx":"Sonraki","se":"Ayarlar","ba":"Geri","sh":"Magaza","ch2":"Karakter","ho":"Ev","ch3":"Sohbet","ra":"Sira"}}

def show_popup(title, text):
    box=BoxLayout(orientation='vertical', padding=15, spacing=10)
    box.add_widget(Label(text=text, color=(0.3,0.2,0.6,1)))
    btn=Button(text="OK", size_hint_y=0.3, background_normal='', background_color=(0.4,0.9,0.6,1), color=(1,1,1,1))
    pop=Popup(title=title, content=box, size_hint=(0.85,0.6))
    btn.bind(on_release=pop.dismiss)
    box.add_widget(btn)
    pop.open()

class W(Screen):
 def __init__(self,d,**k):
  super().__init__(**k); self.d=d; self.b()
 def b(self):
  self.clear_widgets(); fl=FloatLayout()
  with fl.canvas.before: Color(1,0.85,0.95,1); Rectangle(pos=(0,0), size=(1000,1000))
  box=BoxLayout(orientation='vertical', padding=10, spacing=5, size_hint=(0.9,0.9), pos_hint={'center_x':0.5,'center_y':0.5})
  t=T[self.d['lang']]
  box.add_widget(Label(text=f"{t['w']}\nMAHNOOR CANDY\n15000 Levels\n{t['c']}: {self.d['country']} Music ON", bold=True, color=(0.6,0.2,0.7,1), size_hint_y=0.2))
  box.add_widget(Label(text=t['c'], bold=True, color=(0.4,0.2,0.6,1), size_hint_y=0.05))
  cg=GridLayout(cols=2, spacing=3, size_hint_y=0.35)
  for c in C:
   b=Button(text=c, font_size=9, background_normal='', background_color=(0.4,0.9,0.6,1) if c==self.d['country'] else (1,1,1,1), color=(0.3,0.2,0.5,1), bold=True)
   b.bind(on_release=lambda x, cc=c: self.setc(cc))
   cg.add_widget(b)
  box.add_widget(cg)
  box.add_widget(Label(text=t['l'], bold=True, color=(0.4,0.2,0.6,1), size_hint_y=0.05))
  lg=BoxLayout(spacing=3, size_hint_y=0.1)
  for l in L:
   b=Button(text=l, font_size=8, background_normal='', background_color=(0.6,0.8,1,1) if l==self.d['lang'] else (1,1,1,1), bold=True)
   b.bind(on_release=lambda x, ll=l: self.setl(ll))
   lg.add_widget(b)
  box.add_widget(lg)
  st=Button(text=t['s']+" - Music ON", bold=True, size_hint_y=0.12, background_normal='', background_color=(0.4,0.9,0.6,1), color=(1,1,1,1))
  st.bind(on_release=lambda x: self.start_game())
  box.add_widget(st)
  fl.add_widget(box); self.add_widget(fl)
 def setc(self,c):
  SOUND.play("click"); self.d['country']=c; self.b()
 def setl(self,l):
  SOUND.play("click"); self.d['lang']=l; self.b()
 def start_game(self):
  SOUND.play("click")
  if self.d['music']: SOUND.music_on()
  self.manager.current='map'

class M(Screen):
 def __init__(self,d,**k):
  super().__init__(**k); self.d=d
 def on_enter(self): self.b()
 def b(self):
  self.clear_widgets(); fl=FloatLayout(); t=T[self.d['lang']]
  top=BoxLayout(size_hint=(1,0.08), pos_hint={'y':0.92}, padding=4, spacing=4)
  top.add_widget(Label(text=f"Lives {self.d['lives']}/30", bold=True, color=(1,0.2,0.4,1), font_size=10))
  top.add_widget(Label(text=f"{self.d['coins']} {t['co']}", bold=True, color=(0.6,0.4,0.1,1), font_size=10))
  top.add_widget(Button(text=t['se'], size_hint_x=0.3, font_size=9, background_normal='', background_color=(0.3,0.7,1,1), on_release=lambda x: setattr(self.manager, 'current', 'settings')))
  fl.add_widget(top)
  scroll=ScrollView(size_hint=(1,0.82), pos_hint={'y':0.1})
  ml=FloatLayout(size_hint=(1,None), height=2000)
  with ml.canvas.before: Color(0.7,0.95,0.7,1); Rectangle(pos=(0,0), size=(1000,2000))
  pos=[]
  for i in range(1,26):
   x=120 + (i%4)*150; y=1900 - i*70; pos.append((x,y))
  with ml.canvas: Color(0.5,0.8,1,1)
  for i in range(len(pos)-1): Line(points=[pos[i][0], pos[i][1], pos[i+1][0], pos[i+1][1]], width=3)
  for i,(x,y) in enumerate(pos,1):
   col=(1,0.9,0.3,1) if i==self.d['level'] else (0.4,0.9,0.6,1) if i<=self.d['level'] else (0.6,0.6,0.6,0.8)
   btn=Button(text=f"{i}", size_hint=(None,None), size=(55,55), pos=(x-27,y-27), background_normal='', background_color=col, bold=True)
   if i<=self.d['level']: btn.bind(on_release=lambda x, lvl=i: self.go(lvl))
   else: btn.disabled=True
   ml.add_widget(btn)
  scroll.add_widget(ml); fl.add_widget(scroll)
  bottom=BoxLayout(size_hint=(1,0.1), pos_hint={'y':0}, spacing=2, padding=2)
  # WORKING BUTTONS NOW
  shop=Button(text=t['sh'], font_size=8, background_normal='', background_color=(0.8,0.6,0.9,1))
  shop.bind(on_release=lambda x: show_popup("SHOP", f"Welcome to Shop!\nCoins: {self.d['coins']}\nBuy Lives: 10 Coins\nBuy Power: 20 Coins\nCountry: {self.d['country']} Music"))
  chars=Button(text=t['ch2'], font_size=8, background_normal='', background_color=(0.8,0.6,0.9,1))
  chars.bind(on_release=lambda x: show_popup("CHARACTERS", "3 Cute Girls:\n1. Lollipop Girl - Pink\n2. Star Wand Girl - Purple\n3. Heart Girl - Cute\nSelect in Settings!"))
  home=Button(text=t['ho'], font_size=8, background_normal='', background_color=(1,1,1,1))
  home.bind(on_release=lambda x: setattr(self.manager, 'current', 'welcome'))
  chat=Button(text=t['ch3'], font_size=8, background_normal='', background_color=(0.8,0.6,0.9,1))
  chat.bind(on_release=lambda x: show_popup("CHAT", "Friends Online: 5\nPakistan: 2\nIndia: 1\nUSA: 2\nChat Coming Soon!"))
  rank=Button(text=t['ra'], font_size=8, background_normal='', background_color=(0.8,0.6,0.9,1))
  rank.bind(on_release=lambda x: show_popup("RANK", f"Your Rank: #{self.d['level']}\nCoins: {self.d['coins']}\nLevel: {self.d['level']}/15000\nTop 1: Mahnoor 15000"))
  for b in [shop,chars,home,chat,rank]: bottom.add_widget(b)
  fl.add_widget(bottom); self.add_widget(fl)
 def go(self,lvl):
  SOUND.play("click")
  self.d['level']=lvl; self.d['wins']=0; self.manager.current='game'

class G(Screen):
 def __init__(self,d,**k):
  super().__init__(**k); self.d=d
 def on_enter(self):
  self.is_complete=False; self.b()
 def b(self):
  self.clear_widgets(); fl=FloatLayout()
  with fl.canvas.before: Color(0.95,0.85,0.98,1); Rectangle(pos=(0,0), size=(1000,1000))
  box=BoxLayout(orientation='vertical', padding=10, spacing=6, size_hint=(1,1))
  t=T[self.d['lang']]
  top=BoxLayout(size_hint_y=0.08, spacing=3)
  top.add_widget(Label(text=f"{t['lv']} {self.d['level']}/15000", bold=True, color=(0.4,0.2,0.6,1), font_size=9))
  top.add_widget(Label(text=f"{t['li']} {self.d['lives']}", bold=True, color=(1,0.2,0.4,1), font_size=9))
  top.add_widget(Label(text=f"{self.d['coins']} {t['co']}", bold=True, color=(0.6,0.4,0.1,1), font_size=9))
  top.add_widget(Button(text=t['se'], size_hint_x=0.25, font_size=8, background_normal='', background_color=(0.3,0.7,1,1), on_release=lambda x: setattr(self.manager, 'current', 'settings')))
  box.add_widget(top)
  box.add_widget(Label(text=f"{t['tg']} {self.d['wins']}/{self.d['target']} - Music {self.d['country']} Playing", bold=True, color=(0.3,0.2,0.6,1), size_hint_y=0.06, font_size=9))
  self.fb=Label(text=t['ch'], font_size=24, bold=True, color=(0.7,0.2,0.8,1), size_hint_y=0.18)
  self.rs=Label(text=f"{self.d['country']} Music ON", color=(0.3,0.3,0.6,1), size_hint_y=0.06, font_size=9)
  box.add_widget(self.fb); box.add_widget(self.rs)
  btns=BoxLayout(size_hint_y=0.2, spacing=8)
  for name,col in [("Rock",(0.6,0.8,1,1)),("Paper",(1,0.8,0.9,1)),("Scissor",(0.7,1,0.7,1))]:
   b=Button(text=name, background_normal='', background_color=col, color=(0.3,0.2,0.5,1), bold=True)
   b.bind(on_release=lambda x,c=name: self.play(c))
   btns.add_widget(b)
  box.add_widget(btns)
  self.next=Button(text=t['nx'], size_hint_y=0.12, background_normal='', background_color=(0.3,0.9,0.5,1), color=(1,1,1,1), bold=True)
  self.next.bind(on_release=self.nxt)
  self.next.disabled=True; self.next.opacity=0
  box.add_widget(self.next)
  back=Button(text=t['ba'], size_hint_y=0.08, background_normal='', background_color=(0.7,0.7,1,1), font_size=11, on_release=lambda x: setattr(self.manager, 'current', 'map'))
  box.add_widget(back)
  fl.add_widget(box); self.add_widget(fl)
 def play(self, player):
  if self.is_complete: return
  comp=random.choice(["Rock","Paper","Scissor"])
  win=(player=="Rock" and comp=="Scissor") or (player=="Scissor" and comp=="Paper") or (player=="Paper" and comp=="Rock")
  t=T[self.d['lang']]
  if player==comp:
   self.fb.text="DRAW!"; self.rs.text=f"Both {comp}"; self.d['streak']=0; SOUND.play("click")
  elif win:
   self.d['wins']+=1; self.d['streak']+=1; self.d['coins']+=15; self.rs.text=f"{t['wi']} +15 Coins - Music {self.d['country']}"
   if self.d['streak']>=5: self.fb.text="EXCELLENT!!!"
   elif self.d['streak']>=3: self.fb.text="PERFECT!!"
   elif self.d['streak']>=2: self.fb.text="GOOD!"
   else: self.fb.text="WIN!"
   SOUND.play("win")
  else:
   self.d['lives']-=1; self.fb.text="OOPS!"; self.rs.text=f"{t['lo']} Comp: {comp}"; self.d['streak']=0; SOUND.play("lose")
  if self.d['wins']>=self.d['target']:
   self.is_complete=True; self.fb.text="LEVEL COMPLETE!"; self.next.disabled=False; self.next.opacity=1; self.d['coins']+=50; SOUND.play("win")
 def nxt(self,*a):
  SOUND.play("click")
  if self.d['level']>=15000: return
  self.d['level']+=1; self.d['wins']=0; self.is_complete=False; self.d['target']=5 if self.d['level']%100==0 else 3
  if self.d['lives']<=0: self.d['lives']=5
  self.b()

class S(Screen):
 def __init__(self,d,**k):
  super().__init__(**k); self.d=d
 def on_enter(self): self.b()
 def b(self):
  self.clear_widgets()
  box=BoxLayout(orientation='vertical', padding=10, spacing=6)
  t=T[self.d['lang']]
  box.add_widget(Label(text=f"{t['se']} - {self.d['country']} Music", bold=True, color=(0.4,0.2,0.6,1), size_hint_y=0.1))
  cg=GridLayout(cols=2, spacing=3, size_hint_y=0.5)
  for c in C:
   b=Button(text=f"{c}", font_size=9, background_normal='', background_color=(0.4,0.9,0.6,1) if c==self.d['country'] else (1,1,1,1), color=(0.3,0.2,0.5,1), bold=True)
   b.bind(on_release=lambda x, cc=c: self.setc(cc))
   cg.add_widget(b)
  box.add_widget(cg)
  lg=BoxLayout(spacing=3, size_hint_y=0.12)
  for l in L:
   b=Button(text=l, font_size=8, background_normal='', background_color=(0.6,0.8,1,1) if l==self.d['lang'] else (1,1,1,1), bold=True)
   b.bind(on_release=lambda x, ll=l: self.setl(ll))
   lg.add_widget(b)
  box.add_widget(lg)
  sb=BoxLayout(size_hint_y=0.12, spacing=10)
  sb.add_widget(Label(text="Music", color=(0.3,0.3,0.6,1), font_size=10))
  from kivy.uix.switch import Switch
  sw1=Switch(active=self.d['music'])
  sw1.bind(active=lambda i,v: self.music_toggle(v))
  sb.add_widget(sw1)
  sb.add_widget(Label(text="Sound", color=(0.3,0.3,0.6,1), font_size=10))
  sw2=Switch(active=self.d['sound'])
  sw2.bind(active=lambda i,v: self.d.update({'sound':v}))
  sb.add_widget(sw2)
  box.add_widget(sb)
  back=Button(text=t['ba']+" to Map", size_hint_y=0.1, background_normal='', background_color=(0.9,0.5,0.7,1), color=(1,1,1,1), bold=True, on_release=lambda x: setattr(self.manager, 'current', 'map'))
  box.add_widget(back)
  self.add_widget(box)
 def setc(self,c):
  SOUND.play("click"); self.d['country']=c; self.b()
 def setl(self,l):
  SOUND.play("click"); self.d['lang']=l; self.b()
 def music_toggle(self,v):
  self.d['music']=v
  if v: SOUND.music_on()
  else: SOUND.music_off()

class FinalApp(App):
 def build(self):
  data={'level':2,'wins':0,'streak':0,'coins':225,'lives':29,'target':3,'country':'Pakistan','lang':'English','music':True,'sound':True}
  sm=ScreenManager()
  sm.add_widget(W(data, name='welcome'))
  sm.add_widget(M(data, name='map'))
  sm.add_widget(G(data, name='game'))
  sm.add_widget(S(data, name='settings'))
  # Start directly at game screen like your photo
  sm.current='game'
  return sm

if __name__=="__main__":
 FinalApp().run()
