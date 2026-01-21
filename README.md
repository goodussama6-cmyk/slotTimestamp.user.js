# slotTimestamp.user.js
// ==UserScript==
// @name         BLS Submit Timestamp + Slot Detection
// @namespace    bls
// @version      1.0
// @description  show slot submit timestamp & slot detected timestamp
// @match        https://www.blsspainmorocco.net/MAR/Appointment/SlotSelection*
// @match        https://www.blsspainmorocco.net/MAR/Appointment/ApplicantSelection*
// @grant        none
// @updateURL    https://gist.githubusercontent.com/Oussama/<GIST_ID>/raw/slotTimestamp.user.js
// @downloadURL  https://gist.githubusercontent.com/Oussama/<GIST_ID>/raw/slotTimestamp.user.js
// ==/UserScript==

(function() {
    'use strict';

    // --------------------------------------------------------------------
    // PART 1 — SlotSelection (KEEP AS IS)
    // --------------------------------------------------------------------
    if (location.href.includes("SlotSelection")) {

        let lastClock = "NO CLOCK";

        function getClock(){
            const el = Array.from(document.querySelectorAll("div.btn.btn-primary"))
                            .find(e => /\d{2}:\d{2}:\d{2}\.\d{3}/.test(e.innerText));
            return el ? el.innerText.trim() : "NO CLOCK";
        }

        function display(t){
            let box = document.getElementById("submitClock");
            if(!box){
                box = document.createElement("div");
                box.id = "submitClock";
                box.style.fontFamily = "monospace";
                box.style.fontSize = "20px";
                box.style.fontWeight = "bold";
                box.style.marginTop = "10px";
                box.style.padding = "8px 12px";
                box.style.background = "#e7f1ff";
                box.style.border = "1px solid #007bff";
                box.style.borderRadius = "6px";
                document.querySelector("form")?.prepend(box);
            }
            box.innerText = "Submit at: " + t;
        }

        // ultra fast polling @ ~1ms
        setInterval(()=>{
            const current = getClock();
            if(current !== "NO CLOCK") lastClock = current;
        }, 1);

        // hook click for auto-submit
        const int = setInterval(()=>{
            const btn = document.getElementById("btnSubmit");
            if(btn){
                clearInterval(int);
                const realClick = btn.click;

                btn.click = function(){

                    // IMPORTANT: Save timestamp BEFORE navigation
                    localStorage.setItem("last_slot_submit", lastClock);

                    // optional display for SlotSelection page
                    display(lastClock);

                    return realClick.apply(this, arguments);
                };
            }
        },200);
    }

    // --------------------------------------------------------------------
    // PART 2 — ApplicantSelection (Display stored timestamp)
    // --------------------------------------------------------------------
    if (location.href.includes("ApplicantSelection")) {

        const t = localStorage.getItem("last_slot_submit");
        if(!t) return;

        const int2 = setInterval(()=>{
            const target = document.querySelector("div.d-none.d-sm-block.col-md-5");
            if(target){
                clearInterval(int2);

                const box = document.createElement("div");
                box.style.fontFamily = "monospace";
                box.style.fontSize = "22px";
                box.style.fontWeight = "bold";
                box.style.padding = "10px 15px";
                box.style.marginTop = "10px";
                box.style.background = "#e7f1ff";
                box.style.border = "1px solid #007bff";
                box.style.borderRadius = "6px";
                box.style.color = "#000";
                box.innerText = "Slot Detected at: " + t;

                target.appendChild(box);
            }
        },200);
    }

})();

