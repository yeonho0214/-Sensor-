3-1_arduino_code_실습
====================
## 0. 준비 단계
### 0.1. Jetson nano에 파이썬 3.8 설치하기

1.update & upgrade
<pre>
<code>
sudo apt update
sudo apt upgrade
</code>
</pre>
2. 필요한 패키지 설치
<pre>
<code>
sudo apt install build-essential libssl-dev zlib1g-dev libncurses5-dev libncursesw5-dev libreadline-dev 
libsqlite3-dev libgdbm-dev libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev libffi-dev libc6-dev
</code>
</pre>
3. python3.8 소스코드 받기
<pre>
<code>
cd /
sudo wget https://www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz
</code>
</pre>
4. 압축 풀기
<pre>
<code>
sudo tar -xf Python-3.8.12.tar.xz
cd Python-3.8.12
</code>
</pre>
5. Build
<pre>
<code>
./configure --enable-optimizations
make -j4
</code>
</pre>
6. 마무리
<pre>
<code>
sudo make altinstall
python3.8 --version
</code>
</pre>
7. 가상환경 (중요!!)
<pre>
<code>
python3.8 -m venv myenv                                     
source myenv/bin/activate
</code>
</pre>
>
<pre>
<code>
pip install —trusted-host pypi.org —trusted-host
files.pythonhosted.org pip setuptools
</code>
</pre>

<img src="https://github.com/user-attachments/assets/a3ccebf4-b803-4867-ac58-9d69a208dc82" width="30%" />
<img src="https://github.com/user-attachments/assets/26b1c21e-20ae-406d-9a08-69a20982fff6" width="30%" />
<img src="https://github.com/user-attachments/assets/46911449-f62a-4a53-912b-a141c13b0f5b" width="30%" />



### 0.2. Jetson nano에 jupyter notebook 설치하기

* 앞으로 python3.8 사용할때는 아래 코드를 사용해서 가상환경으로 들어간다. 가상환경 이름: myenv
<pre>
<code>
source myenv/bin/activate
</code>
</pre>
* jupyter notebook 설치
<pre>
<code>
pip install jupyter
</code>
</pre>

<pre>
<code>
pip install openai
</code>
</pre>

<pre>
<code>
Pip install gradio
</code>
</pre>

* jupyter notebook 실행
<pre>
<code>
jupyter notebook
</code>
</pre>
<img src="https://github.com/user-attachments/assets/3fdc497f-537b-4982-8255-538bbfee75af" width="70%" />



### 0.3. Jetson GPIO
1. 가상환경에서 Jetson.GPIO 깔려있는지 확인 -> 숫자가 나와야함 -> 깔려있다면 다음 단계 안해도 됨
<pre>
<code>
source myenv/bin/activate
python3 -c "import Jetson.GPIO as GPIO; print(GPIO.VERSION)"
Deactivate
</code>
</pre>
2. 가상환경 비활성화하고 아래 코드 실행해서 깔려있는지 확인 -> 깔려있다면 다음단계 X
<pre>
<code>
python3 -c "import Jetson.GPIO as GPIO; print(GPIO.VERSION)"
</code>
</pre>
3. 가상환경 비활성화에서도 안깔려있으면 코드 실행 / 깔려있으면 실행 안해도 됨
<pre>
<code>
sudo apt-get update
sudo apt-get install python3-jetson-gpio
</code>
</pre>
4. 가상환경에서만 안되는거면? 아래 코드 실행 -> 다시 1번단계 실행해보기
<pre>
<code>
cp -r /usr/lib/python3/dist-packages/Jetson /home/dli/myenv/lib/python3.8/site-packages/
cp -r /usr/lib/python3/dist-packages/Jetson.GPIO-2.0.17.egg-info /home/dli/myenv/lib/python3.8/site-packages/
</code>
</pre>
<img src="https://github.com/user-attachments/assets/9d17d306-dffd-4aa1-b8c3-772a1507a7c3" width="40%" />
<img src="https://github.com/user-attachments/assets/39e0722c-212d-4479-bc49-b97ddfd1575e" width="40%" />

### 0.4. 아두이노
* 기존 아두이노 삭제
>
    sudo apt remove —purge arduino
    sudo apt autoremove
>
* 1.8.19 버전으로 다운로드: <https://www.arduino.cc/en/software>
* arduino-1.8.19라는 폴더에 압축해제
>
    cd Downloads
    tar -xf arduino-1.8.19-linuxaarch64.tar.xz
>
    cd arduino-1.8.19
    sudo ./install.sh
    sudo usermod -aG dialout $USER
    newgrp dialout
>
    arduino
<img src="https://github.com/user-attachments/assets/40500c61-2513-48a5-a653-2d17640da353" width="80%" />


## 1. 미세먼지 읽어오는 코드

    import Jetson.GPIO as GPIO
    import time
    import math

    def measure_pm25():
        """
        PM2.5 농도를 측정하여 반환하는 함수.

        Args:
            pin (int): 측정에 사용할 GPIO 핀 번호 (BCM 핀 기준).
            sample_time_ms (int): 샘플링 시간 (밀리초 단위).

        Returns:
            float: PM2.5 농도 (ug/m3).
        """

        pin = 8
        sample_time_ms=30000
        # GPIO 초기화
        GPIO.setmode(GPIO.BCM)
        GPIO.setup(pin, GPIO.IN)

        low_pulse_occupancy = 0
        start_time = time.time()

        try:
            # 샘플링 시간 동안 LOW 신호 지속 시간 측정
            while (time.time() - start_time) * 1000 <= sample_time_ms:
                pulse_start = time.time()
                while GPIO.input(pin) == GPIO.LOW:
                    pass
                pulse_end = time.time()

                # LOW 상태 지속 시간 계산
                pulse_duration = (pulse_end - pulse_start) * 1e6  # 마이크로초 단위로 변환
                low_pulse_occupancy += pulse_duration

            # PM2.5 농도 계산
            ratio = low_pulse_occupancy / (sample_time_ms * 10.0)
            concentration = (
                1.1 * math.pow(ratio, 3) - 3.8 * math.pow(ratio, 2) + 520 * ratio + 0.62
            )
            return str(round(concentration,2))

        except Exception as e:
            print(f"Error during measurement: {e}")
            return None

        finally:
            GPIO.cleanup()

    # 예시: 함수 호출
    if __name__ == "__main__":
        pm25_concentration = measure_pm25()
        if pm25_concentration is not None:
            print(f"Measured PM2.5 Concentration: {pm25_concentration:} ug/m3")
        else:
            print("Measurement failed.")
***
> <결과값>
Measured PM2.5 Concentration: 927746.07 ug/m3

## 2. 함수정의
<pre>
<code>
use_functions = [
    {
        "type": "function",
        "function": {
            "name": "measure_pm25",
            "description": "Reads and calculatses the fine dust (PM2.5 and PM10) concentration from the given sensor data"

        }
    }
]
</code>
</pre>
***
## 3. Chat completions
<pre>
<code>
import os
from openai import OpenAI
import json

os.environ['OPENAI_API_KEY'] = 
</code>
</pre>

## 4. Gradio로 GUI 구성하기
<pre>
<code>
def ask_openai(llm_model, messages, user_message, functions = ''):
    client = OpenAI()
    proc_messages = messages

    if user_message != '':
        proc_messages.append({"role": "user", "content": user_message})

    if functions == '':
        response = client.chat.completions.create(model=llm_model, messages=proc_messages, temperature = 1.0)
    else:
        response = client.chat.completions.create(model=llm_model, messages=proc_messages, tools=functions, tool_choice="auto") # 이전 코드와 바뀐 부분

    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls

    if tool_calls:
        # Step 3: call the function
        # Note: the JSON response may not always be valid; be sure to handle errors

        available_functions = {
            "measure_pm25": measure_pm25
        }

        messages.append(response_message)  # extend conversation with assistant's reply

        # Step 4: send the info for each function call and function response to GPT
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)


            print(function_args)

            if 'user_prompt' in function_args:
                function_response = function_to_call(function_args.get('user_prompt'))
            else:
                function_response = function_to_call(**function_args)

            proc_messages.append(
                {
                    "tool_call_id": tool_call.id,
                    "role": "tool",
                    "name": function_name,
                    "content": function_response,
                }
            )  # extend conversation with function response
        second_response = client.chat.completions.create(
            model=llm_model,
            messages=messages,
        )  # get a new response from GPT where it can see the function response

        assistant_message = second_response.choices[0].message.content
    else:
        assistant_message = response_message.content

    text = assistant_message.replace('\n', ' ').replace(' .', '.').strip()


    proc_messages.append({"role": "assistant", "content": assistant_message})

    return proc_messages, text
</code>
</pre>

<pre>
<code>
import gradio as gr
import random


messages = []

def process(user_message, chat_history):

    # def ask_openai(llm_model, messages, user_message, functions = ''):
    proc_messages, ai_message = ask_openai("gpt-4o-mini", messages, user_message, functions= use_functions)

    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="채팅창")
    user_textbox = gr.Textbox(label="입력")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
</code>
</pre>
***
<img src="https://github.com/user-attachments/assets/f4eaa4fa-f9e3-493c-993e-2373d5221776" width="80%" />
