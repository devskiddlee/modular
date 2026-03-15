# modular
A c++ library for modular handling of events made primarly for cheat clients

# Usage
Download the [header file](https://raw.githubusercontent.com/devskiddlee/modular/refs/heads/main/modular.h)

### Events
In Modular there are 2 event types:
- Tick Events
- Render Events

### Tick Events
Tick Events can either be called manually or run in a loop by modular.

#### Register
```cpp
Modular::AddTickEventHandler(Reader::OnTick);
Modular::AddTickEventHandler(Aimbot::OnTick);
Modular::AddTickEventHandler(Misc::OnTick);
Modular::AddTickEventHandler(HUD::OnTick);
Modular::AddTickEventHandler(GameEvents::OnTick);
```
To register a Tick Event Handler you call ```Modular::AddTickEventHandler(...)```,
providing a function with following signature: ```void OnTick(const TickEvent& event)```.
The handlers get called every Tick Event with a ```TickEvent``` struct provided which contains the duration of the last tick in seconds.
```cpp
struct TickEvent {
	float delta_time;
};
```

#### Loop
```cpp
Modular::SetTickCap(100);
Modular::StartTickLoop();
...
Modular::StopTickLoop();
```
You start the Tick Loop using ```Modular::StartTickLoop()``` and stop it using ```Modular::StopTickLoop()```.
You can, at any point, set a tick cap which caps the ticks per second that get run, you do this by calling ```Modular::SetTickCap(nTickCap)```.

#### Delayed Tasks
```cpp
Modular::ScheduleDelayedTask(G::S.triggerbotDelay, "schedule_shoot_fn", [](const TickEvent& event) {
	mouse_event(0x0002, 0, 0, 0, GetMessageExtraInfo());
	Modular::ScheduleDelayedTask(10.f, "release_shoot_btn", [](const TickEvent& event) {
		mouse_event(0x0004, 0, 0, 0, GetMessageExtraInfo());
    Modular::ScheduleDelayedTask(..., "free_shoot_fn", [](const TickEvent& event) {
      free_to_shoot = true;
    });
  });
});
```
While running the tick loop you might want to schedule a task that gets run at a later time.
To do this you call ```Modular::ScheduleDelayedTask(...)``` with following parameters:
<br>```delay```: The delay in Milliseconds
<br>```id```: A cstr that identifies the given task
<br>```fn```: The callback which gets run after the given delay. The ```TickEvent``` parameter passes the info of the tick that calls the function to the callback.

#### Key Events
```cpp
Modular::AddKeyEventHandler(VK_INSERT, [](bool pressed) {
	if (pressed) {
    // button was pressed
	} else {
    // button was released
  }
});
```
While technically being a seperate event type, Key Events get checked and called during tick calls.
To register a key event handler you call ```Modular::AddKeyEventHandler(...)``` with following parameters:
<br>```vk```: The Virtual Key that triggers the callback (either passed in as an ```int``` or ```int*```)
<br>```fn```: The callback which gets run when the key gets pressed. The ```bool``` parameter is ```true``` when the key has been pressed and ```false``` when it has been released.

### Render Events
Render Events can only be called manually.

#### Register
```cpp
Modular::AddRenderEventHandler(Reader::OnRender);
Modular::AddRenderEventHandler(Misc::OnRender);
Modular::AddRenderEventHandler(PlantedC4::OnRender);
Modular::AddRenderEventHandler(ESP::OnRender);
Modular::AddRenderEventHandler(HUD::OnRender);
```
To register a Render Event Handler you call ```Modular::AddRenderEventHandler(...)```,
providing a function with following signature: ```void OnRender(const RenderEvent& event)```.
The handlers get called every Render Event with a ```RenderEvent``` struct provided which contains the duration of the last tick in seconds.
```cpp
struct RenderEvent {
	ImDrawList* drawList;
	float last_draw_time;
};
```
##### ImGui
If you are using ImGui, you can define its path with the ```MD_IMGUI_PATH``` macro
```cpp
#define MD_IMGUI_PATH "path/to/imgui.h"
#include "modular.h"
```
On Default ImGui's path is ```<imgui/imgui.h>```
##### No ImGui
If you are not using ImGui, you can avoid it by defining a macro before including ```modular.h```
```cpp
#define MD_AVOID_IMGUI
#include "modular.h"
```
If you do this, the ```RenderEvent``` struct now only contains the last draw duration.
```cpp
struct RenderEvent {
	float last_draw_time;
};
```

#### Calling Render Events
```cpp
RenderEvent event;
event.drawList = ImGui::GetBackgroundDrawList();
event.last_draw_time = lastFrameTime / 1000.f;
Modular::CallRenderEvent(event);
```
The ```drawList``` member is not present if ```MD_AVOID_IMGUI``` has been defined.
```Modular::CallRenderEvent(...)``` calls every registered Render Event Handler with the passed in event.

### Debugging
#### Speed Debugging
```Modular::GetAverageTickTime()``` returns the average duration of the last few ticks.
```cpp
Modular::EnableTickSpeedDebugging();
Modular::EnableRenderSpeedDebugging();
```
To get specific times, Speed Debugging needs to be enabled by using these functions.
Upon activation, tick and render event handlers get timed and stored as a TimeReport struct.
To get the reports you call ```Modular::GetTickTimeReports(...)``` or ```Modular::GetRenderTimeReports(...)```.
Parameters are a pointer to a ```TimeReport*``` and a ```size_t``` that represent the beginning of the TimeReport array and its size.
```cpp
TimeReport* rep = nullptr;
size_t size = 0;
Modular::GetTickTimeReports(&rep, &size);
for (size_t i = 0; i < size; i++) {
  std::cout << '[' << i << "] : " << rep[i].avgTime << "ms" << '\n';
}
```
```cpp
struct TimeReport {
	float lastTickTimes[8];
	float avgTime;
	size_t currentTickTimeIndex;
};
```
