Debouncing is a technique similar to Throttling. 
It's best described here http://unscriptable.com/2009/03/20/debouncing-javascript-methods/

Popular javascript libraries (jQuery, _.js) provide debouncing helpers. This project aims to provide a similar reusable library for Java. 

There are two parts to this project:

1. The Debouncing framework that provides helper classes for debouncing methods. This framework doesn't have any dependencies and can be quickly reused in any project.
2. A Debounce annotation and matching AOP aspect to debounce methods declaratively. 
	The annotation has been successfully tested with Spring-AOP. 
	
Both use cases are demonstrated in tests. 

### A simple Android example that debounces updates from a seek bar

* clone this project into the directory next to your project

e.g. it should look like this when you're done
```bash
$ ls
MyAwesomeProject
JDebounce
```

* add the dependency to your build.gradle file in the main project (MyAwesomeProject/build.gradle)

... indicates other parts of the file, don't add these!
```
...
dependencies {
    ...
    compile project(':..:JDebounce')
    ...
}
...
```

* implement the debouncer

```java
import ch.arrg.jdebounce.DebounceExecutor;
import ch.arrg.jdebounce.DebouncerStore;

public class MyAwesomeActivity extends android.app.Activity {
  private static final int SEEK_BAR_UPDATE_SPEED = 700;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.awesome);
    initLayout();
  }

  private void initLayout() {
    ((SeekBar) findViewById(R.id.seek_bar)).setOnSeekBarChangeListener(new OnSeekBarChangeListener() {
      @Override
      public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
        // We're only concerned about updates from the user
        if (fromUser) {
          debounceOnProgressChange(progress);
        }
      }
    }
  }

  DebouncerStore<String> store = new DebouncerStore<String>();
  private void debounceOnProgressChange(final int progress) {
    // This name should be unique to this debouncer
    DebounceExecutor exec = store.registerOrGetDebouncer("seek");
    // This runnable should call the debounced method
    exec.debounce(SEEK_BAR_UPDATE_SPEED, new Runnable() {
      public void run() {
        // TODO: debounced method goes here
      }
    });
  }
}
```
