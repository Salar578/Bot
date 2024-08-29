from highrise import *
from highrise.models import *
from asyncio import run as arun
from flask import Flask
from threading import Thread
from highrise.__main__ import *
from emotes import*
import random
import asyncio
import time

class Bot(BaseBot):
    def __init__(self):
        super().__init__()
        self.emote_looping = False
        self.user_emote_loops = {}
        self.loop_task = None
        self.is_teleporting_dict = {}
        self.following_user = None
        self.following_user_id = None
        self.kus = {}
        self.user_positions = {} 
        self.position_tasks = {} 


    async def on_emote(self, user: User, emote_id: str, receiver: User | None) -> None:
      print(f"{user.username} emoted: {emote_id}")
  
    async def on_start(self, session_metadata: SessionMetadata) -> None:
        print("hi im alive?")
        await self.highrise.tg.create_task(self.highrise.teleport(
            session_metadata.user_id, Position(15.50, 0.00, 14.00, "FrontRight")))
             

    async def on_user_join(self, user: User, position: Position | AnchorPosition) -> None:
        await self.highrise.chat(f"خوش اومدی🤍! @{user.username} برای رقصیدن از 1 تا 96 را وارد کنید و برای رقصیدن تصادفی بنویسید دنس ✨برای ایستادن بنویسید 0 یا stop!")
        try:
            emote_name = random.choice(list(secili_emote.keys()))
            emote_info = secili_emote[emote_name]
            emote_to_send = emote_info["value"]
            await self.send_emote(emote_to_send, user.id)
        except Exception as e:
            print(f"Error sending emote to user {user.id}: {e}")
  
    async def on_user_leave(self, user: User):
    
        user_id = user.id
        farewell_message = f"! @{user.username}"
        if user_id in self.user_emote_loops:
            await self.stop_emote_loop(user_id)
        await self.highrise.chat(farewell_message)
  

    async def on_chat(self, user: User, message: str) -> None:
        """On a received room-wide chat."""    

        if message.lower().startswith("kiss"):
          await self.highrise.send_emote("emote-kiss")

        isimler1 = [
            "\n1 - @JimmyBey",
            "\n2 - @YusufSinns",
            "\n3 - @Forzo404",
            "\n4 - @chinarro",
            "\n5 - @izHirry",
        ]
        isimler2 = [
            "\n6 - @syndicatexe",
            "\n7 - @TheSanji",
            "\n8 - @Suriyelicarsafi",
            "\n9 - @uzivice",
            "\n10 - @TBO28",
        ]
        isimler3 = [
            "\n11 - @ITraxy",
            "\n12 - @B4TUESCOB4R",
            "\n13 - @pengumen",
            "\n14 - @avenger08",
            "\n15 - @M4ST1",
        ]
        isimler4 = [
            "\n16 - @Jerey",
            "\n17 - @Batukekocu",
            "\n18 - @ELXAANN",
            "\n19 - @EFUL1M01",
            "\n20 - @iKousei",
        ]
        isimler5 = [
            "\n21 - @u.c.a.r.20",
            "\n22 - @1CcRuuaz",
            "\n23 - @MehmetHKT",
            "\n24 - @aroeye",
            "\n25 - @awelll1",
            "\n26 - @Oguzhainzz"

            # Diğer isimler buraya eklenecek
        ]

        
        if message.lower().startswith("::"):
          await self.highrise.chat("\n".join(isimler1))
          await self.highrise.chat("\n".join(isimler2))
          await self.highrise.chat("\n".join(isimler3))
          await self.highrise.chat("\n".join(isimler4))
          await self.highrise.chat("\n".join(isimler5))
          await self.highrise.chat(f"\n\nÖnerileriniz için @M3RIT")

          


        message = message.lower()

        teleport_locations = {
            "3vip": Position(15.5, 16.0, 17.),
            "kat1": Position(13.5, 11.0, 7.5),
            "zemin": Position(11., 1., 11.),
            "deneme1": Position(random.randint(0, 40), random.randint(0, 40), random.randint(0, 40)),
            "deneme1": Position(random.randint(0, 40), random.randint(0, 40), random.randint(0, 40))
        }

        for location_name, position in teleport_locations.items():
            if message ==(location_name):
                try:
                    await self.teleport(user, position)
                except:
                    print(" ")
                  
        if message.lower().startswith("kick") and await self.is_user_allowed(user):
            target_username = message.split("@")[-1].strip()
            room_users = await self.highrise.get_room_users()
            user_info = next((info for info in room_users.content if info[0].username.lower() == target_username.lower()), None)

            if user_info:
                target_user_obj, initial_position = user_info
                task = asyncio.create_task(self.reset_target_position(target_user_obj, initial_position))

                if target_user_obj.id not in self.position_tasks:
                    self.position_tasks[target_user_obj.id] = []
                self.position_tasks[target_user_obj.id].append(task)

        elif message.lower().startswith("") and await self.is_user_allowed(user):
            target_username = message.split("@")[-1].strip()
            room_users = await self.highrise.get_room_users()
            target_user_obj = next((user_obj for user_obj, _ in room_users.content if user_obj.username.lower() == target_username.lower()), None)

            if target_user_obj:
                tasks = self.position_tasks.pop(target_user_obj.id, [])
                for task in tasks:
                    task.cancel()
                print(f"Breaking position monitoring loop for {target_username}")
            else:
                print(f"User {target_username} not found in the room.")

        if message.lower().startswith("info"):
            target_username = message.split("@")[-1].strip()
            await self.userinfo(user, target_username)


        if message.startswith("+x") or message.startswith("-x"):
            await self.adjust_position(user, message, 'x')
        elif message.startswith("+y") or message.startswith("-y"):
            await self.adjust_position(user, message, 'y')
        elif message.startswith("+z") or message.startswith("-z"):
            await self.adjust_position(user, message, 'z')
              
      
        allowed_commands = ["switch", "degis", "değiş","değis","degiş"] 
        if any(message.lower().startswith(command) for command in allowed_commands) and await self.is_user_allowed(user):
            target_username = message.split("@")[-1].strip()

        
            if target_username not in self.haricler:
                await self.switch_users(user, target_username)
            else:
                print(f"{target_username}  ")

        if                          message.lower().startswith("tp") or message.lower().startswith("tp"):
          target_username =         message.split("@")[-1].strip()
          await                     self.teleport_to_user(user, target_username)
        if await self.is_user_allowed(user) and message.lower().startswith("getir"):
            target_username = message.split("@")[-1].strip()
            if target_username not in self.haricler:
                await self.teleport_user_next_to(target_username, user)
        if message.lower().startswith("--") and await self.is_user_allowed(user):
            parts = message.split()
            if len(parts) == 2 and parts[1].startswith("@"):
                target_username = parts[1][1:]
                target_user = None

                room_users = (await self.highrise.get_room_users()).content
                for room_user, _ in room_users:
                    if room_user.username.lower() == target_username and room_user.username.lower() not in self.haricler:
                        target_user = room_user
                        break

                if target_user:
                    try:
                        kl = Position(random.randint(0, 40), random.randint(0, 40), random.randint(0, 40))
                        await self.teleport(target_user, kl)
                    except Exception as e:
                        print(f"An error occurred while teleporting: {e}")
                else:
                    print(f"'{target_username}'")

        if message.lower() == "full rup" or message.lower() == "full kaç":
            if user.id not in self.kus:
                self.kus[user.id] = False

            if not self.kus[user.id]:
                self.kus[user.id] = True

                try:
                    while self.kus.get(user.id, False):
                        kl = Position(random.randint(0, 29), random.randint(0, 29), random.randint(0, 29))
                        await self.teleport(user, kl)

                        await asyncio.sleep(0.7)
                except Exception as e:
                    print(f"Teleport sırasında bir hata oluştu: {e}")

        if message.lower() == "0" or message.lower() == "stop":
            if user.id in self.kus: 
                self.kus[user.id] = False

        if message.lower().startswith("ceza") and await self.is_user_allowed(user):
            target_username = message.split("@")[-1].strip().lower()

            if target_username not in self.haricler:
                room_users = (await self.highrise.get_room_users()).content
                target_user = next((u for u, _ in room_users if u.username.lower() == target_username), None)

                if target_user:
                    if target_user.id not in self.is_teleporting_dict:
                        self.is_teleporting_dict[target_user.id] = True

                        try:
                            while self.is_teleporting_dict.get(target_user.id, False):
                                kl = Position(random.randint(0, 39), random.randint(0, 29), random.randint(0, 39))
                                await self.teleport(target_user, kl)
                                await asyncio.sleep(1)
                        except Exception as e:
                            print(f"An error occurred while teleporting: {e}")

                        self.is_teleporting_dict.pop(target_user.id, None)
                        final_position = Position(1.0, 0.0, 14.5, "FrontRight")
                        await self.teleport(target_user, final_position)
                    

        if message.lower().startswith("dur") and await self.is_user_allowed(user):
            target_username = message.split("@")[-1].strip().lower()

            room_users = (await self.highrise.get_room_users()).content
            target_user = next((u for u, _ in room_users if u.username.lower() == target_username), None)

            if target_user:
                self.is_teleporting_dict.pop(target_user.id, None)
                

        if message.lower() == "follow" and await self.is_user_allowed(user):
            if self.following_user is not None:
                await self.highrise.chat(" ")
            else:
                await self.follow(user)

        if message.lower() == "ok" and await self.is_user_allowed(user):
            if self.following_user is not None:
                await self.highrise.chat("follow")
                self.following_user = None
            else:
                await self.highrise.chat(" ")
              
        if message.lower().startswith("kick") and await self.is_user_allowed(user):
            parts = message.split()
            if len(parts) != 2:
                return
            if "@" not in parts[1]:
                username = parts[1]
            else:
                username = parts[1][1:]

            room_users = (await self.highrise.get_room_users()).content
            for room_user, pos in room_users:
                if room_user.username.lower() == username.lower():
                    user_id = room_user.id
                    break

            if "user_id" not in locals():
                return

            try:
                await self.highrise.moderate_room(user_id, "kick")
            except Exception as e:
                return


      
        message = message.strip().lower()
        user_id = user.id
      
        if message.startswith(""):
            emote_name = message.replace("", "").strip()
            if user_id in self.user_emote_loops and self.user_emote_loops[user_id] == emote_name:
                await self.stop_emote_loop(user_id)
            else:
                await self.start_emote_loop(user_id, emote_name)
                
        if message == "stop" or message == "dur" or message == "0":
            if user_id in self.user_emote_loops:
                await self.stop_emote_loop(user_id)
                
        if message == "دنس":
            if user_id not in self.user_emote_loops:
                await self.start_random_emote_loop(user_id)
                
        if message == "stop" or message == "0":
            if user_id in self.user_emote_loops:
                if self.user_emote_loops[user_id] == "دنس":
                    await self.stop_random_emote_loop(user_id)
     

        message = message.strip().lower()

        if "@" in message:
            parts = message.split("@")
            if len(parts) < 2:
                return

            emote_name = parts[0].strip()
            target_username = parts[1].strip()

            if emote_name in emote_mapping:
                response = await self.highrise.get_room_users()
                users = [content[0] for content in response.content]
                usernames = [user.username.lower() for user in users]

                if target_username not in usernames:
                    return

                user_id = next((u.id for u in users if u.username.lower() == target_username), None)
                if not user_id:
                    return

                await self.handle_emote_command(user.id, emote_name)
                await self.handle_emote_command(user_id, emote_name)


        for emote_name, emote_info in emote_mapping.items():
            if message.lower() == emote_name.lower():
                try:
                    emote_to_send = emote_info["value"]
                    await self.highrise.send_emote(emote_to_send, user.id)
                except Exception as e:
                    print(f"Error sending emote: {e}")


        if message.lower().startswith("all ") and await self.is_user_allowed(user):
            emote_name = message.replace("all ", "").strip()
            if emote_name in emote_mapping:
                emote_to_send = emote_mapping[emote_name]["value"]
                room_users = (await self.highrise.get_room_users()).content
                tasks = []
                for room_user, _ in room_users:
                    tasks.append(self.highrise.send_emote(emote_to_send, room_user.id))
                try:
                    await asyncio.gather(*tasks)
                except Exception as e:
                    error_message = f"Error sending emotes: {e}"
                    await self.highrise.send_whisper(user.id, error_message)
            else:
                await self.highrise.send_whisper(user.id, "Invalid emote name: {}".format(emote_name))
    
              
        message = message.strip().lower()

        try:
            if message.lstrip().startswith(("uçur")):
                response = await self.highrise.get_room_users()
                users = [content[0] for content in response.content]
                usernames = [user.username.lower() for user in users]
                parts = message[1:].split()
                args = parts[1:]

                if len(args) >= 1 and args[0][0] == "@" and args[0][1:].lower() in usernames:
                    user_id = next((u.id for u in users if u.username.lower() == args[0][1:].lower()), None)

                    if message.lower().startswith("uçur"):
                        await self.highrise.send_emote("emote-telekinesis", user.id)
                        await self.highrise.send_emote("emote-gravity", user_id)
        except Exception as e:
            print(f"An error occurred: {e}")
          
        if message.startswith("rd") or message.startswith("danslar"):
            try:
                emote_name = random.choice(list(secili_emote.keys()))
                emote_to_send = secili_emote[emote_name]["value"]
                await self.highrise.send_emote(emote_to_send, user.id)
            except:
                print(" ")


#Numaralı emotlar numaralı emotlar
  
    async def handle_emote_command(self, user_id: str, emote_name: str) -> None:
        if emote_name in emote_mapping:
            emote_info = emote_mapping[emote_name]
            emote_to_send = emote_info["value"]

            try:
                await self.highrise.send_emote(emote_to_send, user_id)
            except Exception as e:
                print(f"Error sending emote: {e}")


    async def start_emote_loop(self, user_id: str, emote_name: str) -> None:
        if emote_name in emote_mapping:
            self.user_emote_loops[user_id] = emote_name
            emote_info = emote_mapping[emote_name]
            emote_to_send = emote_info["value"]
            emote_time = emote_info["time"]

            while self.user_emote_loops.get(user_id) == emote_name:
                try:
                    await self.highrise.send_emote(emote_to_send, user_id)
                except Exception as e:
                    if "Target user not in room" in str(e):
                        print(f"{user_id} ")
                        break
                await asyncio.sleep(emote_time)

    async def stop_emote_loop(self, user_id: str) -> None:
        if user_id in self.user_emote_loops:
            self.user_emote_loops.pop(user_id)


  
#paid emotes paid emotes paid emote
  
    async def emote_loop(self):
        while True:
            try:
                emote_name = random.choice(list(paid_emotes.keys()))
                emote_to_send = paid_emotes[emote_name]["value"]
                emote_time = paid_emotes[emote_name]["time"]
                
                await self.highrise.send_emote(emote_id=emote_to_send)
                await asyncio.sleep(emote_time)
            except Exception as e:
                print("Error sending emote:", e) 


  
#Ulti Ulti Ulti Ulti Ulti Ulti Ulti

    async def start_random_emote_loop(self, user_id: str) -> None:
        self.user_emote_loops[user_id] = "دنس"
        while self.user_emote_loops.get(user_id) == "دنس":
            try:
                emote_name = random.choice(list(secili_emote.keys()))
                emote_info = secili_emote[emote_name]
                emote_to_send = emote_info["value"]
                emote_time = emote_info["time"]
                await self.highrise.send_emote(emote_to_send, user_id)
                await asyncio.sleep(emote_time)
            except Exception as e:
                print(f"Error sending random emote: {e}")

    async def stop_random_emote_loop(self, user_id: str) -> None:
        if user_id in self.user_emote_loops:
            del self.user_emote_loops[user_id]



  #Genel Genel Genel Genel Genel

    async def send_emote(self, emote_to_send: str, user_id: str) -> None:
        await self.highrise.send_emote(emote_to_send, user_id)
      


    async def on_whisper(self, user: User, message: str) -> None:
        """On a received room whisper."""
        if await self.is_user_allowed(user) and message.startswith(''):
            try:
                xxx = message[0:]
                await self.highrise.chat(xxx)
            except:
                print("error 3")
  
    async def is_user_allowed(self, user: User) -> bool:
        user_privileges = await self.highrise.get_room_privilege(user.id)
        return user_privileges.moderator or user.username in ["XI0S","L.zq","M3RIT"]

#gellllbbb

    async def moderate_room(
        self,
        user_id: str,
        action: Literal["kick", "ban", "unban", "mute"],
        action_length: int | None = None,
    ) -> None:
        """Moderate a user in the 
