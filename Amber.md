# Amber

**`Amber`** это архитектура построенная на Elm и Redux идеях специально для iOS. 

Для того, чтобы приступить к рассказу о ней, нам необходимо настроить проект, чтобы можно было сразу практиковаться. Давайте сделаем это.

### Установка Amber

> **1.** Создаем проект в Xcode
> 
> **2.** Создаем `pod` файл, в котором указываем необходимые нам для работы библиотеки, а именно (`Amber`, `Bond`). Вызываем команду `pod install`.
> 
> **3.** Скачиваем [шаблоны](https://github.com/Anvics/AmberModule) для [`Generamba`](https://github.com/rambler-digital-solutions/Generamba)
> 
> **4.** В папке проекта вызываем команду `generamba setup`, отвечаем на все [вопросы](https://docs.google.com/document/d/1Vs5NINPyQ9PVxTSmcB9fZa55wK2kBXl4eh0X4igb8zM/edit#heading=h.is2ecd1c9omo).
> 
> **5.** Вызываем команду `generamba template install`.
> 
> **6.** В папку с шаблонами копируем шаблоны Amber, которые мы скачали.
> 
> **7.** Готово, чтобы создать модуль, в папке проекта вызываем `generamba gen <Имя модуля> Amber`.

Открываем проект `.xcworkspace`.

### Приступаем к работе с Amber

Если вы создали модуль `Amber`, то можете удалить `ViewController` файл и `Storyboard`, которые были созданы Apple по умолчанию. Откроем папку с модулем и увидим там 6 файлов и папку View.

1. Первый файл `<Имя модуля>State`
2. Второй файл `<Имя модуля>InputOutput`
3. Третий файл `<Имя модуля>Reducer`
4. Четвертый файл `<Имя модуля>Router`
5. Пятый файл `<Имя модуля>Controller`
6. Шестой файл `<Имя модуля>Animator`

Теперь по порядку о каждом файле.

### State

`State` это то место, где у нас хранится состояние всех переменных, которые мы используем в модуле. 

Выглядит это следующим образом.

~~~swift
import Foundation
import Amber

struct TestModuleState: AmberState {

    var description: String { return "" }

    init(data: Void) { }
}
~~~
