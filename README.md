# Документация Web SDK Adcamp

- [О технологии](#technology)
  - [Поддержка совместимости MRAID](#mraid)
  - [Инициализация и жизненный цикл](#lifecycle)
- [Технические требования](#requirments)
  - [Обращение к родительской странице](#req1)
  - [Проверка нативных возможностей](#req2)
  - [Контейнер для креатива](#req3)
  - [Конкатенация скриптов и стилей](#req4)
- [Перечень методов](#methods)
  - mraid.getState()
  - mraid.getPlacementType()
  - mraid.getScreenSize()
  - mraid.getCurrentPosition()
  - mraid.getExpandProperties()
  - mraid.isViewable()
  - mraid.getOrientationProperties()
  - mraid.setOrientationProperties()
  - mraid.addEventListener()
  - mraid.removeEventListener()
  - mraid.expand()
  - mraid.close()
  - mraid.destroy()
  - mraid.useCustomClose()



***
## <a name="technology"></a>О технологии
#### <a name="mraid"></a> Поддержка совместимости MRAID
Технология MRAID ([официальная документация](http://www.iab.net/media/file/IAB_MRAID_v2_FINAL.pdf)) была создана [IAB](http://www.iab.net/) для взаимодействия веб-содержимого и native кода в мобильных приложениях (получение размеров экрана, отслеживание событий etc). WEB SDK Adcamp [наследует большинство ее методов](#methods), позволяя использовать MRAID креативы и в веб-браузерах.
#### <a name="lifecycle"></a> Инициализация и жизненный цикл
Для начала работы с SDK, на странице необходимо подключить загрузчик и указать блоки с параметрами ([подробнее о установке кода](https://github.com/adcamp/web-sdk)). После загрузки в блоке с параметрами будет создан iframe с состоянием loading, до момента окончательной подгрузки креатива. C помощью методов mraid.expand(), mraid.resize() и mraid.close() можно управлять состоянием креатива. Так как вместо предопределенного контейнера в приложении, Web SDK использует iframe для показа креативов, это накладывает некоторые требования к разработчику.

***
## <a name="requirments"></a>Технические требования
#### <a name="req1"></a> Обращение к родительской странице
В целях совместимости в рекламных материалах HTML запрещается обращение к родительской странице (window.parent). Все скрипты должны работать относительно контейнера креатива.

**Неправильно:**
```
window.parent.document.body.append(script);
document.body.append(script);
```
**Правильно:**
```
document.body.getElementById('custom_creative').append(script);
```
Также iframe накладывает ограничения, не свойственные корневому окну, поэтому необходимо удостовериться в корректной работе креатива из third-party фрейма.
* * *

#### <a name="req2"></a> Проверка нативных возможностей
Если креатив планируется использовать для нескольких SDK, необходимо проверять наличие методов свойственных мобильным приложениям. Это касается и поддержки HTML5:
```
function canvasSupport(){
  return !!document.createElement('canvas').getContext; };
  if(!canvasSupport()){
    var fallback = document.getElementById('fallback'); fallback.src = imagePath + 'backup.jpg';
  return;
};
```
* * *
#### <a name="req3"></a> Контейнер для креатива
Креатив должен находится внутри одного элемента с ID и не содержать глобальных тегов head, body, указания доктайпа etc.
* * *
#### <a name="req4"></a> Конкатенация скриптов и стилей
В целях оптимизации времени загрузки, рекомендуется помещать все скрипты и стили в конкатенированные файлы.

**Плохо:**
```
<script type="text/javascript" src="http://example.com/part1.js"></script>
<script type="text/javascript" src="http://example.com/part2.js"></script>
<script type="text/javascript" src="http://example.com/part3.js"></script>
```
**Хорошо:**
```
<script type="text/javascript" src="http://example.com/all.js"></script>
```
* * *
## <a name="methods"></a>Перечень методов

#### mraid.open(target : string)
Совершает открытие цели (target) в новом окне. Для правильного подсчета статистики, не разрешается использовать значение аргумента, отличного от значения переменной clickURL, которая доступна глобально. Метод может вызываться только после совершения целевого действия пользователем (например, клик). 
В DOM вместо явного указания атрибута onclick можно использовать класс "adc_clickarea": в элементы с этим классом добавятся обработчики события, которые вызовут mraid.open(clickURL).
**Неправильно:**
```
<a href="http://example.com" target="_blank">Target</a>
<div onclick="mraid.open('http://example.com')">Target</div>
```
**Правильно:**
```
<a onclick="mraid.open(clickURL)" target="_blank">Target</a>
<div class="adc_clickarea">Target</div>
```

#### mraid.expand()
Разворачивает фрейм на весь экран. Метод не может вызываться из состояния expanded, для возврата в состояние default необходимо использовать close(). Включает триггер expand и меняет состояние креатива на expanded.

#### mraid.resize()
Производит смену размера креатива на основе объекта mraid.getResizeProperties(). Включает триггер resize и меняет состояние креатива на resized Для изменения размера креатива необходимо назначить новые параметры для ресайза и выполнить метод.

**Пример:**
```
  var screenSize = mraid.getScreenSize(); 
  var resizeProperties = { 
    "width": screenSize.width, 
    "height": screenSize.height/2, 
    "offsetX": 0,
    "offsetY": screenSize.height/10, 
    "allowOffscreen": false 
  }
mraid.setResizeProperties(resizeProperties);
mraid.resize(); 
```
#### mraid.close()
Если состояние expanded - возвращает в default, иначе - аналогичен методу mraid.destroy(). Уничтожение рекламы должно происходить только с помощью этого, в противном случае после закрытия баннера останется iframe.

#### mraid.destroy()
Уничтожает фрейм из любого состояния.

#### mraid.getState()
Возвращает текущее состояние баннера (string)
Возвращаемые значения: loading (default), default, expanded, resized, hidden

#### mraid.getPlacementType()
Возвращает тип плейсмента (string)
Возвращаемые значения: inline, interstitial

#### mraid.getScreenSize()
Возвращает объект с размерами экрана (object)
{width: (int), height: (int)}

#### mraid.getCurrentPosition()
Возвращает объект с размерами креатива и его отступами по осям (object)
{x: (int), y: (int), width: (int), height: (int)}

#### mraid.getExpandProperties()
Возвращает объект с настройками expand состояния (object)
{width: (int), height: (int), useCustomClose: (bool), isModal: (bool)}

#### mraid.getResizeProperties()
Возвращает объект с настройками expand состояния (object)
{x: (int), y: (int), width: (int), height: (int)}

#### mraid.isViewable()
Возвращает видимость объекта (bool)

#### mraid.getOrientationProperties()
Возвращает объект с параметрами смены ориентации объекта (bool)

#### mraid.setOrientationProperties(allowOrientationChange: bool, forceOrientation: str)
Назначает параметры смены ориентации объекта

#### mraid.setOrientationProperties({x: int, y: int, width: int, height: int})
Назначает параметры для смены размера креатива

#### mraid.addEventListener(event, handler)
Назначает слушателя события

#### mraid.removeEventListener(event)
Уничтожает слушателя события

#### mraid.useCustomClose(bool)
Если в шаблоне объявлен mraid.useCustomClose(true), то крестик показан не будет, однако область скрытия останется в правом верхнем углу.

