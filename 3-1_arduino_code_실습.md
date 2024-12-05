3-1_arduino_code_실습
====================

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
