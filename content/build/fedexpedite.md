---
title: "fedexpedite"
date: "2025-01-09"
description: "a python based route optimiser and emission calculator for FedEx based on the held-karp algorithm. optimise routes, minimise emissions & maximise efficiency! \n ps: won 1st place in the fedex smart hackathon"
tags: ["dev", "hackathon"]
mermaid: true
---

> a python based route optimiser and emission calculator for FedEx based on the held-karp algorithm. optimise routes, minimise emissions & maximise efficiency!
> ps: won 1st place in the fedex smart hackathon.

<div id="ppt-slider" style="max-width:600px; margin:auto;">
     <img id="slide-img" src="/img/build/fedexpedite/1.png" style="width:100%">
     <br>
     <div style="display: flex; justify-content: center; gap: 10px; margin-top: 10px;">
         <button onclick="prevSlide()">Prev</button>
         <button onclick="nextSlide()">Next</button>
     </div>
</div>
<script>
let slideIndex = 1;
const slides = [
	"/img/build/fedexpedite/1.png",
	"/img/build/fedexpedite/2.png",
	"/img/build/fedexpedite/3.png",
    "/img/build/fedexpedite/4.png",
    "/img/build/fedexpedite/5.png",
    "/img/build/fedexpedite/6.png",
    "/img/build/fedexpedite/7.png",
    "/img/build/fedexpedite/8.png",
    "/img/build/fedexpedite/9.png",
    "/img/build/fedexpedite/10.png",
    "/img/build/fedexpedite/11.png",
    "/img/build/fedexpedite/12.png",
    "/img/build/fedexpedite/13.png",
    "/img/build/fedexpedite/14.png",
    "/img/build/fedexpedite/15.png",
    "/img/build/fedexpedite/16.png",
    "/img/build/fedexpedite/17.png"
];
function showSlide(n) {
	const img = document.getElementById('slide-img');
	if (n < 1) slideIndex = slides.length;
	else if (n > slides.length) slideIndex = 1;
	else slideIndex = n;
	img.src = slides[slideIndex - 1];
}
function prevSlide() { showSlide(slideIndex - 1); }
function nextSlide() { showSlide(slideIndex + 1); }
</script>

> [github repo](https://github.com/1rvinn/FedExpedite)



