---
title: "bubble"
date: "2025-08-06"
description: "your intelligent native gui guide"
tags: ["ai", "dev"]
mermaid: true
---

---

<div style='text-align:center;'>
    <h3 style="color: #23affd;"> // the final thing </h3>

check it out here: https://github.com/1rvinn/bubble_v2


<iframe width="560" height="315" src="https://www.youtube.com/embed/WdP8bOORbTs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
<br>
<br>
</div>

---

<div style='text-align:center;'>
    <br>
    <h3 style="color: #23affd;"> // building it out - the journey</h3>
</div>
for the gemma 3n hackathon by google deepmind, i intend on making a local ai based screen overlay helper that allows people to call it to command wherever and whenever they face an issue in their day to day tasks. 

the vision is pretty similar to google’s gemini live screen streaming, but with an added layer of ui - a screen overlay that displays the key troubleshooting steps/guide along with proper highlighting of the important elements to be accessed in the process.

what this possibly could look like is people in photoshop, autocad or matlab, being stuck with finding a particular feature, unable to move ahead. now, instead of laboriously looking for a solution across google, youtube, stack-exchanges and chatgpt, they access our tool via a shortcut, describe their issue in natural language and get the requisite solution along with visual and auditory cues.

this could further transition into the opportunity of integrating this with core apps such as the ones aforementioned and provide real time assistants (this could be a business opportunity right here).

---

[2 july 2025] [1700 hrs]

i looked at gemini live and also ran it via the terminal using their api. it is pretty much the barebones version of what i wanna achieve, still there is a lot to be done.

this is how it works using their api:

- uses pyaudio to capture audio
- records screen/video frames using cv2
- then uses this to create a session with gemini live and the streaming starts
    
    ```python
    client.aio.live.connect(model, config)
    ```
    

[https://github.com/google-gemini/cookbook/blob/main/quickstarts/Get_started_LiveAPI.py](https://github.com/google-gemini/cookbook/blob/main/quickstarts/Get_started_LiveAPI.py)

^^ this is the code for the api implementation

now, the issue is, this wont work with local models, gemma 3n in this case. why? because gemma does not support this streaming feature which is being leveraged above.

and local ai, irrespective of the hackathons instructions, is important. you wouldnt want to share everything you do on your pc with google servers sitting around the globe in california.

---

[2 july 2025] [2300 hrs]

cool so gemma 3n does not accept video inputs. and well, even if it did, it’d have been a nightmare for them to be processed locally, while screen capture is on. doesnt accept mp3 too (and prolly other audio formats also).

i’ll try sending the final snapshot. dont know how effective that is going to be, but is prolly the only option i have. 

also, the image and the user prompt will have to be pretexted by some context as to what the application is and, and idk what.

for the prompt, i will take in the user audio - use a speech to text.

---

[3 july 2025] [1300 hrs]

the above approach is more or less okay but definitely not the best. ideally, what i would want to achieve is the screen being recorded continuously. everything that has been recorded before the user’s questions goes to an visual llm for understanding, which is then summarised, ie, a summary of everything that has happened since the starting/last prompt till now. the summary should then be passed along with the latest frame and the user’s prompt.

the common thing between both the two approaches is the latter part, ie, the last frame and the user prompt. 

so for the first iteration, i’ll use the first approach - basic context, last frame and user prompt. for the second iteration - summary of all activities till now, last frame and user prompt.

let the hacking begin!

---

### version 0.1

---

[3 july 2025] [1307 hrs]

so now, i need to think of a pipeline for screen capture at the final frame and user input.

let me just have text input for starters.

the flow should look like this:

<div style='text-align:center;'>
    {{< mermaid >}}
        flowchart LR;
        A[shortcut]--->|opens overlay|B[app interface]--->|user entered prompt|D[llm]
        B--->|last frame screenshot|D--->|response|B
    {{< /mermaid >}}
</div>

visual highlighting of the steps involved might be complex. but koina, kar lenge.

---

[1800 hrs]

found a way to run multimodal gemma 3n locally, using hugging face transformers.

here’s the documentation: https://ai.google.dev/gemma/docs/core/huggingface_inference

the issue here is, i do not have much experience with hugging face and especially local models.

what i’ve understood till now is, i have to login via a huggingface token, create a pipeline and then pass the img, text to the pipeline. 

upon doing the above, it gives an error

```bash
OSError: We couldn't connect to 'https://huggingface.co' to load the files, and couldn't find them in the cached files.
Check your internet connection or see how to run the library in offline mode at 'https://huggingface.co/docs/transformers/installation#offline-mode'.
```

fixed - there was an issue with the hf token i had initialised. created a new one and got it done.

---

[4 july 2025] [0015 hrs]

so i have been able to come up with a very very barebones version (version 0.0000001):

- it uses the `pyautogui` library to take a screenshot whenever the script is run,
- `pipeline` from hugging face’s `transformers` library to create a pipeline of `gemma-3n-E2B-it` model,
- and `tkinter` for the gui.
- code
    
    ```python
    import tkinter as tk
    from tkinter import scrolledtext, messagebox
    import pyautogui
    import tempfile
    from PIL import Image, ImageTk
    from transformers import pipeline
    from huggingface_hub import login
    import threading
    
    HF_TOKEN = ""  
    MODEL_ID = "google/gemma-3n-E2B-it"
    
    login(HF_TOKEN)
    pipe = pipeline(
        "image-text-to-text",
        model=MODEL_ID,
        device='cpu'
        )
    
    import os
    
    def capture_screenshot() -> str:
        temp_dir = tempfile.gettempdir()
        screenshot_path = os.path.join(temp_dir, "gemma_screenshot.png")
        screenshot = pyautogui.screenshot()
        screenshot.save(screenshot_path)
        return screenshot_path
    
    def generate_response(prompt: str, image_path: str) -> str:
        try:
            img = Image.open(image_path)
            print("Image loaded successfully")
        except Exception as e:
            raise RuntimeError(f"Failed to open image: {e}")
        chat_prompt = [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {"type": "image", "image": img}
                ]
            }
        ]
        print("Chat prompt created successfully")
        try:
            output = pipe(chat_prompt)
            print("Inference completed successfully")
        except Exception as e:
            raise RuntimeError(f"Failed to generate response: {e}")
        return output[0]['generated_text']
    
    class ScreenshotGemmaApp(tk.Tk):
        def __init__(self):
            super().__init__()
            self.title("gemma hack")
            self.geometry("300x900")
            self.screenshot_path = capture_screenshot()
            # Show screenshot preview
            try:
                img = Image.open(self.screenshot_path)
                img = img.resize((300, 200))  # Resize for preview
                self.tk_img = ImageTk.PhotoImage(img)
    
                self.preview_label = tk.Label(self, image=self.tk_img)
                self.preview_label.pack(pady=(10, 0))
            except Exception as e:
                print("⚠️ Failed to show preview:", e)
            
            self.prompt_box = tk.Text(self, height=5, font=("Arial", 12))
            self.prompt_box.pack(padx=10, pady=(20, 10), fill="x")
    
            self.submit_button = tk.Button(self, text="Submit", font=("Arial", 12), command=self.on_submit)
            self.submit_button.pack(padx=10, pady=10)
    
            self.output_box = scrolledtext.ScrolledText(self, font=("Arial", 12))
            self.output_box.pack(padx=10, pady=(10, 20), fill="both", expand=True)
    
        def on_submit(self):
            prompt = self.prompt_box.get("1.0", tk.END).strip()
            if not prompt:
                messagebox.showwarning("Missing Prompt", "Please enter a prompt.")
                return
    
            self.submit_button.config(state="disabled")
            self.output_box.delete("1.0", tk.END)
            threading.Thread(target=self._run_inference_thread, args=(prompt,), daemon=True).start()
    
        def _run_inference_thread(self, prompt: str):
            try:
                print("starting inference")
                response = generate_response(prompt, self.screenshot_path)
                print("response received")
                self.output_box.insert(tk.END, response)
                print("response:")
                print(response)
                print("end of response")
            except Exception as e:
                messagebox.showerror("Error", str(e))
            finally:
                self.submit_button.config(state="normal")
    
    if __name__ == "__main__":
        app = ScreenshotGemmaApp()
        app.mainloop()
    ```
    

on running it, it wasnt giving any response. so i added a couple of lines to help me debug.

and guess what, the code is absolutely correct, down to the last inch, it is google to blame. 

the model is running slow, like insanely slow. i tried switching over to my gpu by setting `device` equal to `0`, and boom my computer went for a crash. it used something called an MPS, not CUDA ofc because my dearest gpu wasn’t manufactured in nvidia furnaces. so the error was due to my memory capacity not being enough.

```bash
RuntimeError: MPS backend out of memory (MPS allocated: 5.73 GB, other allocations: 384.00 KB, max allowed: 9.07 GB).
Tried to allocate 3.75 GB on private pool. Use PYTORCH_MPS_HIGH_WATERMARK_RATIO=0.0 to disable upper limit for memory allocations (may cause system failure).
```

i went back to cpu then, and after an eternity (8 mins) i have now recd the following response:

![image.png]( /img/build/bubble/image.png)

i had also added a cap on the tokens, for now, to make it run faster. and yes, 8 mins was with a token limit of a mere 128. 

i think there would be a fix to this problem. the very reason google came up with gemma 3n was so it could run on edge devices including mobile phones, which, even the best ones, arent even 30% as beefy as my mac (not saying that ts is great).

google’s edge ai app works on androids and does image inference. i will look into it in the morning and see how they do it. secondly, i’ll revisit this [repo](https://github.com/OminousIndustries/Gemma3n-TTS) i used as reference earlier. third, i’ll look into the documentation again, and maybe what other people are doing for this.

> i may have to later pivot to this not being truly local (while still using gemma 3n for hackathon purposes) and even further to a better, sota cloud model.
> 

here are the tasks for morning now:

- [ ]  figure out how to run it faster
    - [ ]  go through google edge ai
    - [x]  go through gemma documentation
    - [x]  go through this [repo](https://github.com/OminousIndustries/Gemma3n-TTS) (talked about it above) and its corresponding yt vid
    - [ ]  chat gpt in case nothing works

tasks for later

- [ ]  look into cluely’s (not open source so fake cluely’s) source code

1. using the code given in gemma documentation:
    - the only difference in my current code and the one given in gemma’s documentation is this one line:
        
        ```python
        torch_dtype=torch.bfloat16
        ```
        
        i have added this to the pipeline args. lets see how it does now.
        
        dont think it’s making much of a difference. taking more than 10 secs so not there. will update the final time when i receive the response.
        
        also, for some reason the response is still cut off at the end.
        
2. using the code in this [repo](https://github.com/OminousIndustries/Gemma3n-TTS)
    - until now, i have called the model using a pipeline. now, using the above repo as reference, i’ll call it in a different way and see if it helps.
    - running into many errors.

i donot understand what the issue is. i know it can be fixed, i know that for a fact. but how is something i havent been able to figure out yet. i know for sure that it’s hidden in plain sight.

ok, fukkit. truly locla is hard to achieve, atleast for now. i ll chnage it to online inference.

---

[7 july 2025][0036 hrs]

very long time since i last updated this blog or whatever this is. so i’ve been running it on online inference for now, using the gemini-2.0-flash-exp, which i plan on changing to 2.5-flash atleast.

i also vibecoded a slick looking ui; i tried integrating the concepts of glasmorphism, taking inspiration from apple (*good artists copy, great artists steal (jobs))*. surely isnt as goated as apple’s liquid glass but it does it’s job to some extent. here’s how it looks:

![image.png]( /img/build/bubble/image1.png)

![image.png]( /img/build/bubble/image2.png)

there are a few ui changes im still to make:

- [ ]  allow repositioning
- [x]  answer streaming isnt smooth
- [ ]  corners do not have the same curvature
- [ ]  a copy button for code in the answer box
- [x]  an icon for enter
- [ ]  a settings hamburger menu
- [ ]  drastic change: possibly come up with some completely new interface; take inspiration from apple intelligence maybe (oml apple is so goated all my inspirations come from them)

honestly, while going through all the ui stuff, i was loving it. i have always had a knack for design; kinda ocd-ish sometimes, especially when symmetry is involved. and oh reading about glassmorphism, neumorphism, such an interesting, creative pursuit design is. also read about scandinavian design, just beautiful.

i resonate deep with steve when he says - *it’s technology, married with liberal arts, married with the humanities, that yields us the results that make our hearts sing.*

ok now, on the technology side of things. the backend is shitty. there is absolutely no system instruction, i added some debugging lines that are still being printed. and some other issues too. so do the following:

- [ ]  add system instructions
- [x]  get rid of the debugging statements
- [ ]  the screenshot is clicked on each enter, ie, it also has the widget in it; need to change it to be clicked before the widget is shown

— i think up till here will complete most part of this app, then i shall put all focus into doing the visual guide part - the most important bit of this.

---

[7 july 2025][1826 hrs]

the screenshot issue hasnt been fixed. rn what is happening is that the screenshot is clicked at the press of ‘enter’. which is fine from the user’s perspective because the screenshot contains the latest version of what’s on the screen. but is an issue as the screenshot also contains the ask bar, and incase of a followup, also the answer. which makes it hard to see what is behind it on the actual screen. 

i looked at what other apps are doing:

horizon - clicks the ss right after the tool shortcut is called, ie, before the chat box appears. 

glass - works a lot better. it takes updates real time, and also stays consistently over all desktops. also, the responses are a lot better. ig that is the difference between a fun project and a startup.

after trying around a variety of things, i’ve finally settled on making the widget disappear for a while for the screenshot to be captured and then the answer be displayed.

> this isnt the most efficient approach. for the final version, the guide, i plan on making a different ui and taking inspiration from [cheatingdaddy](https://github.com/sohzm/cheating-daddy) to ensure the above in a more streamlined way.
> 

now, i’ll just make it ready for deployment. put in on github and focus on the other bits.

---

[8 july 2025][1700 hrs]

i have pushed it to github. there are a few issues with it, but i have asked a friend to make the changes required to make it fully deployable while i focus on the main bit — the actual model.

---

[8 july 2025][1701 hrs]

i need to firstly analyse how agents work. 

as of now, i have come up with the following pipeline:

<div style='text-align:center;'>
    {{< mermaid >}}
        graph TD;
        screen --> |screenshot or live screencast|omniparser --> |identifies, maps key ui elements|llm
        prompt --> llm --> |action to be performed|visual_cues -->|user| action --> |real time feedback|screen
    {{< /mermaid >}}
</div>

however, what is not clear is how the action and feedback loop works, what exactly the llm outputs when asked the prompt — whether it gives the entire process’ task list (improbable) or gives the first step(s) which is(are) most likely to be correct and once that’s done with, reanalyses the screen and then gives the next one(s). 

what i am thinking would be the best is having a list of broad steps to be outputted first, followed by detailed descriptions for each step one by one, being displayed after the previous one has been executed. 

i think creating an agent is easier than creating a guide as now, people have autonomy to do things, which means, one ill step and the ai gets confused and starts malfunctioning. 

ok let it be. i’ll study more about agents and then see where this goes.

---

[8 july 1800 hrs]

- [x]  check out browser-use
- [x]  check out warm wind
- [x]  check out omni parser

![image.png]( /img/build/bubble/image3.png)

**^^warmwind**

![image.png]( /img/build/bubble/image4.png)

**^^skyvern**

**some yt video:**

[https://colab.research.google.com/drive/1GV4VzhfI8l2uEBm2H9hQ2fs12_iFiYlQ?usp=sharing#scrollTo=x1Edd6dsflaa](https://colab.research.google.com/drive/1GV4VzhfI8l2uEBm2H9hQ2fs12_iFiYlQ?usp=sharing#scrollTo=x1Edd6dsflaa), [https://www.youtube.com/watch?v=Qnp4PQTE1Ag](https://www.youtube.com/watch?v=Qnp4PQTE1Ag)

**os atlas:**

also a library like omniparser, but this returns the pixel coords of the gui element.

[https://github.com/OS-Copilot/OS-Atlas](https://github.com/OS-Copilot/OS-Atlas)

**browser_use:**

![image.png]( /img/build/bubble/image5.png)

---

[9 july 2025][1615 hrs]

i have tried to understand how different computer use agents work and have linked a few repos, screenshots and descriptions above of the ones i found useful.

the pipeline i had made above, wasnt very off. the area’s where it was off, were actually the areas i was confused in. so i’ll make the requisite changes, and let the hacking begin!

<div style='text-align:center;'>
    {{< mermaid >}}
        graph TD;
        user-->|keyboard shortcut|A[user interface] --> |user prompt|B[planner llm]
        A-->|screenshot*|B
        B-->|task plan|C[task list]-->|single task|D[llm]
        A-->|screenshot*|D-->|detailed action description + button/field indices|E[visual cue]
        D-->|pixel coords of action spot|E[visual cue]-->A
        D-->|update|C
    {{< /mermaid >}}
</div>

+feedback loop to be added

*screenshots pass through omniparser

![image.png]( /img/build/bubble/145e733e-62f7-4fdb-9326-59b17d4e0132.png)

^^ easier to understand. i still love pen an pencil over these plastic keycaps.

validator - success, inaction, failure. 

here’s a written version of what i have thought:

```markdown
1. the user calls the app using a shortcut, types their issue (prompt) on the interface
2. the app takes a screenshot, gives it to omniparser which annotates the image to index and tag interactive ui elements
3. the annotated screenshot and prompt goes to the planning llm (llm #1) which devises a list of tasks to be performed to achieve the goal. each tasks include 1 action at max. (for eg: for export to pdf the tasks are - go to file menu, go to export, choose pdf, save)
4. these tasks then go one by one to llm #2 along with a screenshot of the current state of the screen. the decides what action is to be done to complete the task (eg: for 'go to export' - click 'export' button) and also gives the coordinates of the button.
5. this description (click 'export' button) and coordinates is used to highlight the respective button and show it to the user.
6. the app waits for user action. once done, it moves forward to sending the next task to llm#2 and this continues till all the tasks are done
```

ok so for the first version, i’m gonna let cursor do it. i need to understand how these various libraries work and get an idea of how it will all play together. i know the code wont be neat and debugging would be a nightmare for me cuz i haven’t coded it out. but i think it’ll do well for starters, to make me understand the technicalities of it and then later i can code it out with my bare hands.

- vib(code)ing
    
    this is the task list cursor came up with after i gave my pipeline:
    
    - technical task list
        
        ### **1. Electron Overlay App**
        
        - [ ]  Set up Electron app with cross-platform build (Mac/Windows).
        - [ ]  Implement global keyboard shortcut listener.
        - [ ]  Create overlay UI for prompt input and step-by-step guidance.
        - [ ]  Implement overlay rendering for highlights and descriptions (transparent, click-through except for UI).
        - [ ]  Ensure overlay can be shown/hidden and does not interfere with normal app usage.
        
        ### **2. Screenshot Capture**
        
        - [ ]  Implement cross-platform screenshot capture (Node.js native modules or external tools).
        - [ ]  Ensure screenshot is of the correct display (multi-monitor support).
        
        ### **3. Omniparser Integration**
        
        - [ ]  Integrate with Omniparser (Python or API) to send screenshots and receive annotated UI elements.
        - [ ]  Parse and store Omniparser output for downstream use.
        
        ### **4. LLM #1: Task Planning**
        
        - [ ]  Integrate with Gemini API (or other LLM) for planning.
        - [ ]  Send prompt + annotated screenshot, receive ordered list of atomic tasks.
        - [ ]  Parse and store task list.
        
        ### **5. LLM #2: Action & Element Localization**
        
        - [ ]  For each task, send current screenshot + task to LLM #2.
        - [ ]  Receive action description and UI element coordinates.
        - [ ]  Parse and store action/coordinates.
        
        ### **6. Visual Guidance & User Interaction**
        
        - [ ]  Render highlight around UI element at given coordinates.
        - [ ]  Display action description in overlay.
        - [ ]  Track user actions (mouse clicks, keyboard events) to detect when the correct action is performed.
        - [ ]  Handle edge cases (user clicks wrong element, window focus changes, etc.).
        
        ### **7. Task Progression Logic**
        
        - [ ]  Manage task queue and progression.
        - [ ]  On correct user action, advance to next task.
        - [ ]  On completion, show success message and dismiss overlay.
        
        ### **8. Cross-Process Communication**
        
        - [ ]  Set up IPC (inter-process communication) between Electron (Node.js) and Python (Omniparser, LLM calls).
        - [ ]  Ensure robust error handling and fallback for communication failures.
        
        ### **9. Cross-Platform Packaging & Deployment**
        
        - [ ]  Package the app for Mac and Windows (Electron Builder, etc.).
        - [ ]  Bundle Python dependencies and ensure they run on both platforms (PyInstaller, virtualenv, etc.).
        - [ ]  Handle permissions (screen recording, accessibility APIs) for both OSes.
        
        ### **10. Configuration & Extensibility**
        
        - [ ]  Configurable keyboard shortcut.
        - [ ]  Modular architecture for swapping out LLMs or UI parsers.
        - [ ]  Logging and error reporting.
        
    
    ^^ looks pretty robust, i’ll let it start coding.
    
    ok this is f-ing my brains inside out. it feels like a blackbox, i dont understand a thing and the only thing i can do is plead like a begger to have cursor solve the errors. yet, despite the errors im gonna let it continue. so that i atleast know how its trying to integrate all of various things and then i’ll start myself.
    
    ok so the final thing isnt working due to a hell lot of errors, but here is a technical report i asked cursor to make, so my understanding about what it did in the mvp is bettered.
    
    [report - bubble_v2.pdf]( /img/build/bubble/report_-_bubble_v2.pdf)
    

---

[11 july 2025][2222 hrs]

so, last 1.5 ish days, i have been learning js. and i think i have understood it’s basics well. on second thought, i shouldve have spent this much time, the real output has pretty much been 0, as i thought of it to be pretty similar apart from the basic syntax level differences. also, at this point i think, if you have a decent hold over one programming language, it shouldnt be hard to understand others and a more effecient way to go about it would be just to build a project and learn on the fly.

- [x]  im just gonna go through the basics of electron really quick, and then get started with coding.

ok, some basics covered, rest will be dealt with by referring to the documentation.

here’s a great github repo having a ton of boilerplates, tutorials and demo code:

[https://github.com/sindresorhus/awesome-electron](https://github.com/sindresorhus/awesome-electron)

this could be referred to for screenshots/sharing:

[https://github.com/hokein/electron-screen-recorder](https://github.com/hokein/electron-screen-recorder)

- [x]  next, i need to cover asyncio
- [ ]  and backend hosting

also then, i need to look into how cheating daddy works, majorly focusing on:

- [ ]  how are screenshots captured? is it a live stream (using gemini live) or screenshots when the prompt is asked
- [ ]  how do they make it resilient to screensharing

---

[13 july 2025][1800 hrs]

time to get my fingertips dirty, let the hacking begin!

- screenshot parsing
    
    i tried object detection via gemini as mentioned in their documentation but it didn’t work well. so i’ll just try setting up omniparser.
    
    for now, i’ll just run it off hf transformers, later i might set up an hf inference end point and call it via cloud.
    

---

[17 july 2025][1118 hrs]

i have pretty much wasted the last 3-4 days. need to lock in now.

- screenshot parsing
    
    i tried integrating omniparser but no luck. need to do the following, if it still doesn’t go through after this, then we skip to a different model.
    
    - [x]  ask claude for help
        
        ^gives the same old shit
        
    
    i’ve given omniparser a lot of time, but no avail. i even tried to run the entire code by cloning the repo but even that refused to work. so now, im looking to switch over to something else.
    
    - [x]  look into browser-use’s screenshot parsing mech
        
        ^^they use playwright which only works for browsers so cant be implemented
        
    
    ok nvm, omniparser worked. i was running into a variety of different module errors and despite multiple tries, i wasnt able to figure it out, but at the end, i prevailed.
    
    for now, it runs on gradio and o boy gradio is amazing. i can literally use the live link to run omniparser through any device while using my laptop for processing. also, i can use it as an api pretty easily.
    
    however, there are a few issues with this currently:
    
    1. the process takes a lot of time, 30 to 300 secs for images with less and more icons respectively. it isnt the detection algorithm that takes this much time but the caption model instead that takes up most of the time. the former takes less than a second.
        
        the model being used to generate captions is `florence2` or `blip2` which are vision language models, so ofc are bound to take time.
        
    2. i tried running it using the gradio api hosted on hf spaces by microsoft itself, which is returning the result quick, however i am not able to access the image being generated by it. this is what it returns: `'/private/var/folders/60/w....wgn/T/gradio/2...adfa/image.webp'` which means it’s just returning the local path on the server and not the image.
    
    it is imperative to solve these things, but later. for now, i’ll code rest of the app and then finally find a fix to the above, prolly by hosting it somewhere - [fly.io](http://fly.io) or hf spaces.
    

---

[20 jul 2025][1450 hrs]

i’ll start building the planner llm. just a regular gemini 2.5 flash call along with the screenshot being sent. 

i tested it out by sending a screenshot of the chatgpt interface without any annotations and gave this prompt `how do i start a new chat and upload a photo to it?` . here’s the response i got, pretty happy with it for now.

![image.png]( /img/build/bubble/image6.png)

next, i ll work on making the second llm - the action selector. 

the prompt was `Click the '+' button next to the 'Ask anything' text input field.` and it correctly gave me the right icon to be selected.

![image.png]( /img/build/bubble/image7.png)

next steps:

- [x]  create a screenshot capturer
- [x]  combine planner and action selector

one thing i’ve realised is that the captions being generated aren’t very precise and in fact making the action_selector select the wrong element. 
secondly the caption generation model is the reason for such slow responses. i am thinking to let it go, and my hypothesis is that the action_selector will only get more proficient in doing so. however, i do need to test this hypothesis.

ok, now how do i go about removing the captions? in the model i run locally, it is pretty easy since i can play with the code. however, since the model is being called using an api endpoint here, i can’t possibly change the code and neither do i see an option to disable captions being generated.
so maybe i’ll just deploy it locally and use it as an api (possible with gradio)

i have been able to remove the captioning and it takes way lesser than what it used to earlier, with only the annotation happening right now. however, the icon selection is still a little haywire.

before fixing that, i think a couple of other things need to be done:

- [x]  make the planner llm only give out broad descriptions not precise ones (exact button presses not needed)
- [x]  make the action selector llm decide the specifics of the atomic tasks to be performed

the above plan didnt work out, since one task of the planner’s plan can possibly contain multiple actions to be performed. so, i’ve added an atomic generator model that takes in tasks, one by one from the plan, and divides them into actions to be performed based on the screenshot input

so they new pipeline looks as follows

![Mermaid Chart - Create complex, visual diagrams with text. A smarter way of creating diagrams.-2025-07-25-093021.svg]( /img/build/bubble/Mermaid_Chart_-_Create_complex_visual_diagrams_with_text._A_smarter_way_of_creating_diagrams.-2025-07-25-093021.svg)

this is working way better.

next, i need to

- [x]  figure out a way to get the image returned as well
    
    ^^ for this, i changed the api’s output from returning a pil image to the base64 string of that image
    
- [x]  integrate the cue and automatic screenshots
    
    ^^ i have used the keystroke `ctrl+shift+0` to click a screenshot before every step be converted to atomic steps.
    
- [x]  create 2 versions:
    - [x]  one with the planner
    - [x]  one without the planner - only the atomic generator deciding what is to be done on the basis of current state, end goal and previous actions

i think the second version is bound to work better. currently the planner is working on foresight. it is predicting what must be done without having access to the current state. it just takes the initial state and decides the plan. in the case without the planner, the current step is only decided on the basis of the current state, the user intent and current history.
barring the foresight problem, there is also the issue of inefficiency. currently we are employing two models just to decide on the atomic tasks that need to be completed as opposed to one in case the planner is dropped.

hence, i feel its imperative to make the second version the main one.

---

the second version is up and running, and it works pretty damn well. way better than the first/planner one.

there are a few improvements yet to be made:

- [ ]  integrate a validator that verifies whether the previous action has been completed successfully or the user messed up
- [ ]  need to be very specific on which button to click in case of multiple buttons with the same label

i’ll do the above later. for now, i’ll focus on getting the frontend ready.

the frontend will be made using electron js, exact designs for which i havent figured yet.

https://www.electronjs.org/docs/latest

^^ keeping the documentation link here for easy access.

---

so i’ve been able to come up with a raw frontend. not the most aesthetically pleasing app, however, it suffices for the hackathon submission atleast.

![image.png]( /img/build/bubble/image8.png)

however, there still are a few issues with this, which i need to fix:

- [x]  instead of image , upload base 64 upload to omniparser
- [x]  colors and aesthetics improve
- [x]  make sure bounding boxes are complete /neat and adjust the vertical alignment in the bounding box
- [ ]  bounding box moves /shifts if user scrolls in between. can we fix that -
A) make the process very fast
B) control the webpage (not relevant now)
- [x]  integrate a validator that verifies whether the previous action has been completed successfully or the user messed up
- [x]  need to be very specific on which button to click in case of multiple buttons with the same label
- [x]  giving the user flexibility to choose/fill, in case of forms.
- [x]  ctrl+shift+0 for next step ctrl+shift+1 for retry
- [ ]  improve latency
    - [ ]  change gemini calls also to base64
- [x]  remove the drop shadow at the center of the screen
- [x]  rename ‘processing’ messages
- [x]  disappearance on click
- [x]  clickthough not errorless
- [ ]  rephrase task descriptions
- [x]  completed message
- [x]  make it windows compatible
- [x]  doesnt enter typing mode on first open, fix that
- [x]  remove the settings button
- [x]  press crtl+shift+0 for next step
- [x]  fix history handling
- [x]  thinking box visible in screenshots
- [x]  remove ctrl+shift+f ctrl /
- [ ]  make click global
- [x]  description being cutoff
- [ ]  add glow to border
- [x]  fix ‘input focused’ wording and positioning
- [x]  task completed positioning - vertical and horizontal both
- [x]  scrolling
- [x]  instead of the bounding box, make the description box emerge from the place/button that needs to be interacted with, like a comment box.
- [x]  ctrl+shift+0 not working properly
- [ ]  vertical offset in windows
- [x]  entire pipeline should restart when task gets completed

pretty much ready for the submission. 

here is the final pipeline being used:

<div style='text-align:center;'>
    {{< mermaid >}}
        flowchart TD;
        A[user prompt]-->D[atomic task generator]
        C[current screen state]-->D
        B[task history]-->D
        D-->|next atomic task to be executed|E[element selector]
        C-->F[omniparser]-->|annotated image|E-->|element coordinates|G[gui/frontend]
        D-->|task description|G-->|execution success/failure|B
    {{< /mermaid >}}
</div>

here are a few snapshots:

![image.png]( /img/build/bubble/image9.png) 

![image.png]( /img/build/bubble/image10.png)

![image.png]( /img/build/bubble/image11.png)

![image.png]( /img/build/bubble/image12.png)

![image.png]( /img/build/bubble/image13.png)

![image.png]( /img/build/bubble/image14.png)

---

pushing to github:

- [x]  create 2 repos:
    - [x]  bubble_v2
    - [x]  bubble_omni (for the omniparser backend)
- [x]  check requirements.txt
- [x]  readme
    - [x]  banner
    - [x]  short description
    - [x]  video demo
    - [x]  pipeline
    - [x]  how to operate
        - [x]  ctrl/cmd+shift+g to call the app
        - [x]  ctrl/cmd+shift+0 for the next step
        - [x]  ctrl/cmd+shift+1 to retry
    - [x]  how to run locally - instructions to run locally
        - [x]  venv, install reqs at backend/requirements.txt
        - [x]  env, add hf token, gemini api key
        - [x]  run the omni backend, get endpoint url, update in backend/omni_api
            - [x]  venv, install reqs
            - [x]  login using hf token - hugging
        - [x]  npm install, npm 
---
### done!