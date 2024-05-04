# diva-android fix
## InputValidation2URISchemeActivity.java
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/5ef814d8-59a3-4ce4-acd1-646d66a4d50e)

CWE-20: Improper Input Validation

Т.к. отсутствует реализация проверки ввода, можно ввести путь до располодения локального файла, и приложение его выдаст.

  ![image](https://github.com/Din0saur0s/SDL2/assets/70744702/2b5023c4-35a6-4eff-b313-39c1edba258e)

```{r}
 public void get(View view) {
        EditText uriText = (EditText) findViewById(R.id.ivi2uri);
        WebView wview = (WebView) findViewById(R.id.ivi2wview);
        if (isFileUrl(uriText.getText().toString()))
            break;
        else
            wview.loadUrl(uriText.getText().toString());
    }
```
