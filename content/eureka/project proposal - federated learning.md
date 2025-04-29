---
title: 'project proposal - federated learning'
date: "2025-04-29"
description: 'project proposal for ai club'
tags: ["ai", "cysec"]
---

> this project aims at exploring a novel framework for training neural networks - federated learning.

#### brief about federated learning:
traditionally, local availability of training data has been an important pre-requisite for training ml models. however, this methodology has a variety of road blocks associated with it. the prime ones being lack of democratisation in model creation and lower development in fields with sensitive data like finance and healthcare. federated learning aims to solve these challenges by introducing an alternate way of training models - instead of data travelling to the model and the model getting trained, the model travels to data sources (silos, personal devices), gets trained and publishes the updates back to the central model.
this enhances the privacy of ai based systems since the data has to never leave the device. furthermore, this means that people can have full control of their data and yet contirbute tot he development of ai, ie, now proprietary and sensitive data can also be used for training (both pre and post) purposes. this solves the problem of only big data companies being able to train big models, thus leading to more democratization in the field. secondly, it also promotes ai developments in the sensitive data fields like finance and healthcare, which traditionally have been underdeveloped due to privacy concerns.

![federated learning](https://www.dailydoseofds.com/content/images/2023/11/federated-gif.gif)

- a comic by google on the topic: https://federated.withgoogle.com/ 
- an interactive visualisation covering the basics: https://pair.withgoogle.com/explorables/federated-learning/
- an in depth overview: https://queue.acm.org/detail.cfm?id=3501293, https://arxiv.org/pdf/2107.10976
- a small writeup by me on the importance of FL: https://1rvinn.github.io/eureka/fed/

---

> this project will be an amalgamation of ai and security. focusing in depth on the following topics:
> 1. how models are trained [A]
> 2. working of neural nets [A]
> 3. comparision between traditionally trained nets vs FL trained ones
> 4. networking between devices and a central server [S]
> 5. privacy preserving methodologies and the mathematics involved [S]
> 
> A - ai focused topics, S - security focused topics

---
#### milestones to be achieved:
- building the basis (a good part of this can be covered in the app phase) [3 weeks]
	- gain an idea about federated learning (only a high level overview)
		- key steps involved:
			- client selection
			- model distribution
			- update upload
			- aggregation, etc.
		- [understanding privacy considerations](https://www.nist.gov/itl/applied-cybersecurity/privacy-engineering/collaboration-space/blog-series/privacy-preserving):
			- model update attacks
			- trained model attacks
		- current applications in the industry, future prospects
- start off by implementing it on ml models, using already existing frameworks like [flower](https://flower.ai/)/[nvidia flare](https://developer.nvidia.com/flare) [2 weeks]
	- understanding the library
	- implement on a basic ml model
- delving deeper
	- understanding fl in greater detail [3 months]
		- comprehensively understand the maths behind it
		- exploring different training methods like FedAvg, FedSGD etc.
		- security considerations
			- secure aggregation
			- differential privacy
			- homomorphic encryption
		- modeling federated learning from scratch
			- coding the process from scratch, including all the above stated steps ('key steps involved')
			- implementing >=1 privacy preserving strategies (secure agg, diff priv., homo enc)
			- facilitating networking between edge devices and a central server
	- implementation:
		- implement on the following networks (subject to change on the basis of our findings above): [3 months]
			- CNNs
			- [SLMs](https://arxiv.org/abs/2203.09943)
			- implement using existing libraries first [to be displayed in RC] and later from scratch.
			- compare their accuracies with that of normally trained neural nets [to be displayed in RC]
		- look into fine-tuning llms for particular tasks using FL
			- [FLoRA](https://arxiv.org/abs/2409.05976), https://arxiv.org/abs/2102.00875
			- this could be one of the first steps towards making specialised chatbots (possibly even agents) trained on personal private data (great financial use case).
	- further improvements 
		- coming up with solutions to the bottlenecks identified 
		- a few unique approaches include
			- [federated pruning](https://www.alphaxiv.org/abs/2209.06359v1)
			- [federated reconstruction](https://arxiv.org/pdf/2102.03448)
			- federated dropout
- future prospects
	- publishing our own library for federated learning - simplifying tasks like secure aggregation, differential privacy etc.
	- creating an agent specialized for a particular task - financial analysis, tax filing agents.
	- collaborating with hospitals to develop privacy preserving diagnostic models. [link](https://aibusiness.com/verticals/intel-and-upenn-to-use-federated-ai-for-privacy-preserving-brain-tumor-research#:~:text=Instead%20of%20moving%20data%20to,into%20a%20single%2C%20larger%20model)
	- ^ similar collaborations with financial institutions

