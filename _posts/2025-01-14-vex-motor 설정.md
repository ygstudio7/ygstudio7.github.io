# 모터 설정



아래와 같이 encoder 를 E_MOTOR_ENCODER_DEGREES로 설정하면, 이동거리를 쉽게 계산할 수 있다.

그럼, 이동 거리 `/pi D * angle/360' 으로 계산할 수 있다. tick으로는 이값을 계산할 수 없고, tick -> degree 변환을 거처야 한다

그리고, 문서에는 다음과 같이 설정후, move()아닌 move_velocity()같은 함수를 쓸때 이 설정이 매우 쓰기 좋다고 되어있다.

`V5 Motors should be configured before use in your code. Configuration options like the gearset and **encoder** units are important to address first thing in your user program to ensure that functions like [motor_move_velocity](https://pros.cs.purdue.edu/v5/api/c/motors.html#motor-move-velocity) will work as expected.`



```c++
initialize.cpp¶
#define MOTOR_PORT 1

void initialize() {
  pros::Motor drive_left_initializer (MOTOR_PORT, E_MOTOR_GEARSET_18, true, E_MOTOR_ENCODER_DEGREES);
}
opcontrol.cpp¶
#define MOTOR_PORT 1

void opcontrol() {
  pros::Motor drive_left (MOTOR_PORT);
  // drive_left will have the same configuration as drive_left_initializer
}
```



# PID

PID에 관해서는 아래 문서를 참고하라고 기술되어 있다.

`For further reading material on the algorithms that create these profiled movement, see [Mathematics of Motion Control Profiles](https://pdfs.semanticscholar.org/a229/fdba63d8d68abd09f70604d56cc07ee50f7d.pdf) for the [Feedforward](https://en.wikipedia.org/wiki/Feed_forward_(control)) control, and [George Gillard’s PID Explanation](http://georgegillard.com/documents/2-introduction-to-pid-controllers) for the [feedback](https://en.wikipedia.org/wiki/Control_theory#PID_feedback_control) control.`



참고: https://pros.cs.purdue.edu/v5/tutorials/topical/motors.html?highlight=encoder