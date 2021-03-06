project_path: /web/_project.yaml
book_path: /web/fundamentals/_book.yaml
description: Компоновка – это процесс, который сводит воедино прорисованные части страницы для отображения на экране

{# wf_updated_on: 2015-03-19 #}
{# wf_published_on: 2000-01-01 #}

# Используйте свойства, вызывающие только компоновку, и контролируйте количество слоев {: .page-title }

{% include "web/_shared/contributors/paullewis.html" %}


Компоновка – это процесс, который сводит воедино прорисованные части страницы для отображения на экране

### TL;DR {: .hide-from-toc }
- Для достижения анимационного эффекта изменяйте свойства transform и opacity.
- Перемещайте движущиеся элементы на отдельные слои с помощью will-change или translateZ.
- Не используйте слишком много слоев, они занимают память и требуют управления.


В этой области есть два ключевых фактора, которые влияют на производительность страницы: количество слоев, которыми необходимо управлять, и свойства, которые используются для анимации.

## Для достижения анимационного эффекта изменяйте свойства transform и opacity
В самой производительной версии конвейера пикселей отсутствует перерасчет макета и прорисовка. В ней выполняется только изменение компоновки:

<img src="images/stick-to-compositor-only-properties-and-manage-layer-count/frame-no-layout-paint.jpg"  alt="Конвейер пикселей без перерасчета макета или прорисовки.">

Чтобы добиться этого, необходимо будет изменять только те свойства, которые могут обрабатываться исключительно компоновщиком. На сегодня таких свойств только два: **transform** и **opacity**:

<img src="images/stick-to-compositor-only-properties-and-manage-layer-count/safe-properties.jpg"  alt="Свойства, которые можно анимировать без перерасчета макета или прорисовки.">

Помните, что для использования свойств transform и opacity, элемент, для которого вы изменяете эти свойства, должен находиться на _собственном слое_. Чтобы создать слой, необходимо переместить на него элемент, о чем мы поговорим далее.

Note: Если вы беспокоитесь о том, что не сможете в своих анимациях использовать только эти свойства, познакомьтесь с <a href='http://aerotwist.com/blog/flip-your-animations'>принципом FLIP</a>, который, возможно, поможет вам отказаться от использования более затратных свойства и ограничиться только изменением переходов и прозрачности

## Перемещайте элементы, которые планируете анимировать, на отдельные слои

Как уже упоминалось в разделе "[Упрощение прорисовки и сокращение количества областей прорисовки](simplify-paint-complexity-and-reduce-paint-areas)", элементы, которые планируется анимировать, следует переносить на отдельные слои (в пределах разумного, не злоупотребляйте этим!):


    .moving-element {
      will-change: transform;
    }
    

Либо, для старых браузеров или тех, которые не поддерживают свойство will-change:


    .moving-element {
      transform: translateZ(0);
    }
    

Таким образом браузер получает предупреждение о грядущих изменениях. Он же, в зависимости от того, что планируется изменить, может подготовиться, например создать отдельные слои.

## Управляйте слоями и избегайте слишком большого их количества

Иногда, зная, что слои зачастую позволяют повысить производительность, возникает соблазн разместить все элементы страницы на собственных слоях, примерно вот так:


    * {
      will-change: transform;
      transform: translateZ(0);
    }
    

Это способ иносказательно заявить, что вы хотите перенести на собственный слой каждый элемент на странице. Проблема заключается в том, что для каждого создаваемого слоя требуется память и управление, а это все не бесплатно. На самом деле, на устройствах с ограниченным объемом памяти влияние на производительность может намного перевесить любые преимущества, связанные с созданием слоя. Текстуры каждого слоя необходимо загружать в графический процессор, поэтому имеются дополнительные ограничения по пропускной способности между графическим и центральным процессорами, а также памяти, которая есть в графическом процессоре.

Короче говоря, **не размещайте элементы на собственных слоях без надобности**.

## Используйте Chrome DevTools для анализа слоев в своем приложении

Чтобы понять, как работают слои в приложении и почему элемент размещен на отдельном слое, необходимо включить средство профилирования прорисовки на шкале времени Chrome DevTools:

<img src="images/stick-to-compositor-only-properties-and-manage-layer-count/paint-profiler.jpg"  alt="Включение средства профилирования прорисовки в Chrome DevTools.">

Затем выполните запись. После завершения записи вы сможете щелкать отдельные кадры, которые находятся между столбцами, обозначающими количество кадров в секунду, и получать подробные сведения:

<img src="images/stick-to-compositor-only-properties-and-manage-layer-count/frame-of-interest.jpg"  alt="Кадр, профилирование которого требуется разработчику.">

После щелчка кадра в подробных сведениях появится новый элемент – вкладка слоев.

<img src="images/stick-to-compositor-only-properties-and-manage-layer-count/layer-tab.jpg"  alt="Кнопка вкладки слоев в Chrome DevTools.">

При нажатии этой кнопки открывается новое представление, в котором можно панорамировать, сканировать и изменить масштаб отображения на всех слоях в течение последнего кадра, а также просматривать причины создания каждого слоя.

<img src="images/stick-to-compositor-only-properties-and-manage-layer-count/layer-view.jpg"  alt="Представление слоев в Chrome DevTools.">

С помощью этого представления можно определить имеющееся количество слоев. Если у вас уходит много времени на компоновку при выполнении таких критически важных с точки зрения производительности действий, как прокрутка или переходы (целевой показатель равен **4–5 мс**), то с помощью приведенной здесь информации можно определить, сколько у вас слоев, почему они были созданы, а затем уже заняться контролем числа слоев в приложении.


