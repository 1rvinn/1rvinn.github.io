---
title: 'privacy, democratization in ai'
date: "2025-01-14"
description: 'on federated learning and homomorphic encryption'
tags: ["ai", "cysec"]
---

>recently, while reading about the recent developments in the field i realised an array of issues that, in my opinion, are being overlooked in the current paradigm:

1. **monopolisation** \
Due to the humongous data requirement for pre-training, we see big-data companies like the 'Googles' and 'Microsofts' of the world having an unfair competitive edge over other smaller corporations or entities in developing such models. This not only means a handful of companies have control over such a quintessential field but also implies:
\
[D] soft power - greater soft power over all AI-based content, which could further transcend into propagating bias through content and controlling widespread opinions about sensitive topics across the entire world.
\
[P] access to user data - with the advent of time, as AI becomes an indispensable part of our lives, these handful corporations will have even greater access to user data across different use cases, making them even more powerful. Not only this but these days, the most popular LLMs use consumer data for further training, while only mentioning this very subtly in their privacy policy. This makes room for a plethora of data security leaks on the cloud where this data is hosted and other data attacks such as prompt injection.
\
[D] by virtue of huge data and state-of-the-art compute requirements, it is only the firms with a rich history in tech and high levels of disposable income to spend, who can establish themselves in this area, making it discriminatorily hard for smaller firms to emerge in this field. This also means a geopolitical issue for 2nd and 3rd world countries being completely dependent upon the 1st world for such foundational models.


2. **climate impact** \
[D] these foundational models require huge loads of compute requirements for pre-training. This means great amounts of energy requirements, so much so that companies like Microsoft have rented complete nuclear reactors to satisfy the energy hunger of their AI programmes. This move is completely counterintuitive to the sustainability efforts being done to reverse climate change.
\
[S] this tradeoff, considering the first set of harms [1] as well, is completely unjustified and will only lead humanity to its doom.


3. **low Levels of Development in the essential fields of Healthcare and Finance** \
[P] since these models require unrestricted access to data to be trained, coupled with the issue that healthcare and financial data being highly private, we haven't seen any major breakthroughs in these areas. AI has immense potential to change the lives of millions provided there is enough sustainable development in these fields, but the only thing that is holding us back is access to highly private data which is a justified concern given the first set of issues [1].

> with all these concerns in mind, I came across this interesting framework called Federated Learning. coupled with a very interesting encryption scheme - homomorphic encryption - it can solve all the above-listed problems. 

**federated learning:** \
In the current paradigm, the data travels to the model, and the model is trained. But what if the model travels to the data for training? That is, there is no transfer of data, therefore no room for data leaks, privacy issues and greater accessibility for smaller companies without data to compete with big giants.
\
This also needs better innovation as the race now changes to creating novel architectures rather than winning with more data. 
\
Additionally, now there is a greater variety of data available to train these models including personal data - healthcare, financial data. This means better foundation models and more innovation in these underdeveloped areas.
\
Also, it means that the model is trained on individual devices, which implies that the computational and energy requirements are divided across devices, thus offloading the burden on the central server.
\
\
![federated learning](/img/crude/fed/fed1.png)
\
\
**homomorphic encryption:**
\
in the current paradigm, this is how data encryption looks like during LLM inference:
\
\
![homomorphic encryption](/img/crude/fed/fed2.png)
\
the inference is done on decrypted data, which leads to a loophole that can be exploited by cyber attackers. Not only this, but it can also lead to other security issues like the LLM learning about personal details and prompt injections as elaborated in [1].
\
homomorphic encryption prevents this by supporting calculations to be made on encrypted data itself, mitigating any data security threats.
\
additionally, being based upon lattice-based cryptography, it is resilient to the rise of quantum computers which pose a threat to existing encryption schemes like the RSA.
\
\
>if you notice, I have segregated the above problems into three categories - [D]: Democratization of AI, [P]: Privacy, [S]: Sustainability.
\
\
these 3 issues are often always overlooked whenever there is a big innovation, but later, people realise the importance of working on these - the same happened in the case of Internet, with heavy spending on cybersecurity, sustainability and privacy being prevalent only these days.
\
\
the above two frameworks can solve all these issues, empowering even a small startup from Maharashtra in India to compete with OpenAI based out of San Francisco. I truly believe this could be a revolutionary step for democratising AI development and boosting security.