# ReconV2.3 Controller (Android APK)

1. Create a new GitHub repo and upload everything in this folder (keep the `.github` folder).
2. Open the repo > Actions > "Build APK" > wait ~4 min.
3. Open the finished run > Artifacts > download `ReconV2-Controller-apk` (zip) > extract `app-debug.apk`.
4. **Uninstall the old Motor Control app first.** Each GitHub build signs with a new debug key, so Android refuses to install over the old one.
5. Send the APK to the phone, allow "install unknown apps", install.
6. Join Wi-Fi `ESP32_Robot_AP` (tap "Stay connected" if Android warns of no internet), open the app, press Connect.

To change the UI: edit `www/index.html` and push again.
You can also open `www/index.html` in a laptop browser that is joined to the robot's Wi-Fi to test without building.

## Panels
- **Drive** – differential joystick (left = M0/J2, right = M2/J4).
- **Servos** – second joystick: X moves servo 1, Y moves servo 2 (rate control, release = hold). Shows angle, target, speed, load, voltage, temperature. Center / torque on-off.
- **Claw** – type degrees, press − or +. Both claw motors (L = M3/J5, R = M1/J3) move by that many degrees using their encoders.
- **IMU** – roll / pitch / yaw (BMI270), attitude ball, raw accel and gyro, zero-yaw button.
- **Encoders** – all four motors, ticks, degrees and counts/s.
- **Battery** – 2S pack voltage and percentage in the header.
- **Settings** – claw CPR/PWM/direction, servo IDs and speed, and a tool to change a servo's ID.

## WebSocket protocol (app → robot)
| message | meaning |
|---|---|
| `{"t":"drive","l":-255..255,"r":-255..255}` | 20 Hz, 300 ms failsafe |
| `{"t":"srv","x":-1000..1000,"y":-1000..1000}` | servo joystick, 20 Hz |
| `{"t":"srvc","a":"center"}` / `{"t":"srvc","a":"torque","on":true}` | servo actions |
| `{"t":"claw","deg":±N}` / `{"t":"clawstop"}` | claw relative move |
| `{"t":"cfg","cpr","pmin","pmax","invL","invR","s1","s2","srate"}` | settings, sent on connect |
| `{"t":"sid","from":1,"to":2}` | change a servo ID (one servo on the bus) |
| `{"t":"enc0"}` / `{"t":"yaw0"}` | reset encoders / yaw |

Robot → app: `{"t":"tel", enc, spd, claw{busy,deg,err}, srv[{id,ok,deg,tgt,spd,load,v,temp}], torque, imu{ok,r,p,y,ax..gz}, bat{v,pct}, fault}` at 10 Hz, plus `{"t":"log","m":...}`.
