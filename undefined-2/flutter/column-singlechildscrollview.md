# Column 과 SingleChildScrollView

강의 예제에서 얻은 깨달음을 정리한다.

1. transaction 이 쌓이면서 MainAxisSize.max 설정인 Column 의 free space 를 넘어서면서 over flow가 발생한다. 플러터는 이를 그릴수 없으니 에러가 난다.
2. 결국 이 Column 의 height 가 위젯들을 감당할 수 없게 되었으므로 Column 자체를 SingleChildScrollView 로 감싸준다.
3. 그러면 Column 은 자신의 부모가 SingleChildScrollView 이므로 화면을 넘어서는 children 까지 그릴 수 있게 된다. 자신 자체가 SingleChildScrollView 자식이라서 자신이 전체 scroll 이 가능하기 때문에 화면을 벗어난 children 까지 그려도 되는 것이다. 여전히 Column 은 MainAxisSize.max 이라서 Free space가 없지만 자신이 scroll이 가능해졌으므로 free space의 제한 없이 그려도 되는 것이다. Column 은 단지 free space의 제한이 없어진 상태일 뿐이다.
4. 내부에 ListView 에 height를 알려줘야하는 이유는 ListView 가 infinite 한 height 를 차지하기 때문에 ListView가 속한 Column 내부에서 얼마의 공간을 차지시켜야할지를 플러터가 모르기 때문에 지정을 해줘야하는 것이다.
