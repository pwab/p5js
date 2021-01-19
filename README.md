<!--
author:   Philipp Wabnitz

email:    philipp.wabnitz@mb.tu-chemnitz.de

version:  0.0.2

language: de

narrator: US English Female

comment:  P5JS Robot Test

import: https://raw.githubusercontent.com/LiaTemplates/p5js/0.0.2/README.md

-->

# DD-Roboter

```js    +settings.js
let w = 640;
let h = 360;
let linecolor = 33;
let L1 = 80; //Länge Glied A0-A
let L2 = 60; //Länge Glied A-TCP
let theta1 = 0; //Nullstellung
let theta2 = 0; //Nullstellung
let translation = p5.createVector(w/2,h/2); //A0 ist in der Mitte des Fensters
let A0 = p5.createVector(0,0);
let a0a = p5.createVector(L1, 0);
let A = p5.createVector(A0.x + a0a.x, A0.y + a0a.y); //A in Nullstellung
let atcp = p5.createVector(L2, 0);
let TCP = p5.createVector(A.x + atcp.x, A.y + atcp.y); //TCP in Nullstellung
let Pos = p5.createVector(TCP.x, TCP.y); //Soll = Ist
```
``` js   +init.js
/// Initialisierung
p5.setup = function() {
  p5.createCanvas(640, 360);
  p5.strokeWeight(10.0);
  p5.stroke(255, 100);
}

/// Zeichnen der Oberfläche
p5.draw = function() {
  p5.translate(640/2, 360/2);
  p5.scale(1,-1);

  // Farben einstellen
  p5.stroke(linecolor, 100);
  p5.background(255);

  // Arbeitsraumgrenzen
  p5.arc(A0.x, A0.y, (L1+L2)*2, (L1+L2)*2, 0, Math.PI * 2);
  p5.noFill();

  // Roboter einzeichnen
  p5.line(A0.x, A0.y, A.x, A.y);
  p5.line(A.x, A.y, TCP.x, TCP.y);
  p5.point(Pos.x, Pos.y);


  // Roboter einzeichnen
  //p5.line(A0.x + translation.x, A0.y + translation.y, A.x   + translation.x, A.y   + translation.y);
  //p5.line(A.x  + translation.x, A.y  + translation.y, TCP.x + translation.x, TCP.y + translation.y);
  //p5.point(Pos.x + translation.x, Pos.y + translation.y);
}

/// Wenn die Maus über die Oberfläche bewegt wird
p5.mouseMoved = function() {
  Pos.x = p5.mouseX - translation.x;
  Pos.y = -p5.mouseY + translation.y;

  moveRobot();
}

/// Neue Stellung vorgeben
function moveRobot() {
  //1. Gelenkwerte ermitteln
  inverseKinematics(Pos.x, Pos.y);

  //2. TCP ermitteln und anfahren
  forwardKinematics(theta1, theta2);
}
```
``` js -kinematics.js
/// Frage: Wenn ich theta1 und theta2 vorgebe, wie lauten dann x und y des TCP?
function forwardKinematics (t1, t2) {
  //A muss als Zwischenpunkt mit berechnet werden, sonst kann es nicht simuliert werden
  //1. Glied 1 drehen
  a0a = p5.createVector(L1, 0).rotate(t1);
  A = p5.createVector(A0.x + a0a.x, A0.y + a0a.y);

  //2. Glied 2 drehen
  atcp = p5.createVector(L2, 0).rotate(t1 + t2);
  TCP = p5.createVector(A.x + atcp.x, A.y + atcp.y);
}


/// Frage: Wenn ich x und y des TCP vorgebe, wie lauten dann theta1 und theta2?

function inverseKinematics(x, y) {
  //1. Berechnen der Strecke C
  let C = Math.sqrt(x*x + y*y);

  //2. Berechnen von Gamma
  let gamma = Math.atan2(y, x);

  // Grenzwertbetrachtung
  if (C > L1 + L2) {
    linecolor = 100;
    theta1 = gamma;
    theta2 = 0;
    //println(millis(),"Punkt außerhalb des Arbeitsraums");
  }
  else if (C < Math.abs(L1 - L2)) {
    linecolor = 100;
    //println(millis(),"Punkt innerhalb des nicht erreichbaren Arbeitsraums");
  }
  else if ((C == 0) && (L1 == L2)) {
    linecolor = 100;
    //println(millis(),"Unendlich viele Lösungen");
  }

  else {
    linecolor = 0;

    // Variantenbetrachtung
    if (C == L1 + L2) {
       theta1 = gamma;
       theta2 = 0;
    }

    //Berechnung von theta2
    theta2 = Math.acos((x*x+y*y-(L1*L1+L2*L2))/(2*L1*L2));


    //Berechnung von theta1
    let delta = Math.acos((L1*L1-L2*L2+C)/(L1*L2*C));
    theta1 = gamma - delta/2; ///???
  }
}
```
@P5.project
