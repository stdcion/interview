## Цель

Необходимо реализовать функцию с сигнатурой `int dxf_system_set_property(const char *key, const char *value)` для
публичного пользовательского API, которая скрывает от пользователя всю работу связанную с созданием изолята,
аттачем/детачем потоков и т.д.

### Исходные данные

- Тип `graal_isolate_t` это инстанс JVM. Считаем, что может существовать только один инстанс JVM в рамках одного процесса.
- Тип `graal_isolatethread_t` это поток который приаттачен к инстансу JVM. Для вызова любой функции в JVM, необходимо
  предоставить `graal_isolatethread_t`.
- Функция `graal_create_isolate` создает инстанс JVM (`graal_isolate_t`).
- Функция `graal_attach_thread` аттачит текущий поток к инстансу JVM, возвращая `graal_isolatethread_t`.
  К одному инстансу JVM, может быть приаттачено несколько потоков. `graal_isolatethread_t` должен использоваться только
  в том потоке, где он был приаттачен.
- Необходимо вызвать `graal_detach_thread` перед завершением потока для предотвращения утечки ресурсов.
- Разрабатываемая функция является оберткой над `dxfg_system_set_property`.

```c++
/*
 * Creates a new isolate.
 * Returns 0 on success, or a non-zero value on failure.
 */
int graal_create_isolate(graal_isolate_t **isolate);

/*
 * Attaches the current thread to the passed isolate.
 * On failure, returns a non-zero value. On success, writes the address of the
 * created isolate thread structure to the passed pointer and returns 0.
 * If the thread has already been attached, the call succeeds and also provides
 * the thread's isolate thread structure.
 */
int graal_attach_thread(graal_isolate_t *isolate, graal_isolatethread_t **thread);

/*
 * Detaches the passed isolate thread from its isolate and discards any state or
 * context that is associated with it.
 * Returns 0 on success, or a non-zero value on failure.
 */
int graal_detach_thread(graal_isolatethread_t *thread);

/**
 * @brief Sets a system property specified by the given key.
 * @param[in] thread A pointer to the runtime data structure for the current thread.
 * @param[in] key The name of the system property to be set.
 * @param[in] value The new value of the system property.
 * @return 0 on success, or a non-zero value on failure.
 */
int dxfg_system_set_property(graal_isolatethread_t *thread, const char *key, const char *value);
```
#### Пример использования (Internal API)
```c++
int main() {
    graal_isolate_t *isolate = nullptr;
    graal_isolatethread_t *thread = nullptr;

    graal_create_isolate(&isolate);
    graal_attach_thread(isolate, &thread);
    dxfg_system_set_property(thread, "key", "value");
    graal_detach_thread(thread);
}
```

#### Задачи

- Разработать дизайн, учитывающий необходимость создания инстанса JVM, а также аттача/детача потоков
  к этому инстансу.
- Обеспечить безопасное и корректное управление ресурсами.

#### Пример использования (Public API)
```c++
int main() {
    dxf_system_set_property("key", "value");
}
```
