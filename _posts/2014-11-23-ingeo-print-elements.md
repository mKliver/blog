---
title: Копирование элементов макета печати ИнГео
---
Работа с макетом печати не очень хорошо освещена в документации к ИнГео, 
что странно, потому как по крайней мере мне показалась реализация макета печати не очень нативной. И это даже на пользовательском уровне, 
например работа с таблицами.
Я же столкнулся с задачей создания многостраничного макета печати.

Прежде всего некоторые детали касамые возможностей макета печати. 
В макет печати можно включить несколько видов объектов(фигур):
-Карта
-Прямоугольник
-Эллипс
-Линия
-Рисунок
-Текст
-Таблица

Вот например макет, состоящий из участка карты и таблицы:
![Simple maket](/blog/img/2014-11-23/simple-maket.jpg "Simple maket")

Теперь займемся клонированием объектов макета. Начнем с простого - Рисунка. 
Нужно взять копируемый объект из макета и затем создать новый, пустой рисунок:
{% highlight c# linenos %}

var sourceImage = (IInPicturePictureFigure)MyLayoutWindow.Figures.Find("imageName");
var targetImage = MyLayoutWindow.Figures.Add(TInPictureFigureType.inftPicture) as IInPicturePictureFigure;
CloneImage(targetImage , sourceImage );

{% endhighlight %}

А вот и сам метод для копирования:
{% highlight c# linenos %}

public static IInPicturePictureFigure CloneImage(IInPicturePictureFigure target, IInPicturePictureFigure source)
 {
	target.Width = source.Width;
	target.Height = source.Height;

	target.Picture = source.Picture;
	target.Stretch = source.Stretch;
	target.HorAlign = source.HorAlign;
	target.VerAlign = source.VerAlign;
	target.TransparentBack = source.TransparentBack;

	target.Pen.Mode = source.Pen.Mode;
	target.Pen.Style = source.Pen.Style;
	target.Pen.WidthInMM = source.Pen.WidthInMM;
	target.Pen.Color = source.Pen.Color;

	target.Brush.HatchColor = source.Brush.HatchColor;
	target.Brush.BackColor = source.Brush.BackColor;
	target.Brush.Transparency = source.Brush.Transparency;
	target.Brush.Style = source.Brush.Style;

	return target;
  }

{% endhighlight %}

Как видите метод копирует свойства объекта из существующего в новый. 
После можно установить положение объекта на макете пользуясь свойствами фигуры **Bottom** и **Left**.

Для остальных объектов макета печати используются аналогичные методы поэтому приведу еще один пример для Прямоугольника:
{% highlight c# %}
public static IInPictureRectFigure CloneRectangle(IInPictureRectFigure target, IInPictureRectFigure source)
{
	target.Width = source.Width;
	target.Height = source.Height;

	target.Pen.Mode = source.Pen.Mode;
	target.Pen.Style = source.Pen.Style;
	target.Pen.WidthInMM = source.Pen.WidthInMM;
	target.Pen.Color = source.Pen.Color;

	target.Brush.HatchColor = source.Brush.HatchColor;
	target.Brush.BackColor = source.Brush.BackColor;
	target.Brush.Transparency = source.Brush.Transparency;
	target.Brush.Style = source.Brush.Style;

  return target;
}
{% endhighlight %}

Немного по другому обстоит дело с Таблицей.
Дело в том, что у этой фигуры есть два набора параметров: стандартные - находятся в левой панели,
 и специальные - находится в контекстном меню в **Редакторе таблицы**.

Стандартные настройки позволяют настроить таблицу в целом, а так же сделать стиль по умолчанию, 
который будет применяться ко всем строкам/столбцам/ячейкам. 
Поэтому если нужно создать таблицу неизвестного в заранее размера, то стиль нужно задавать именно в левой панели.

![Default properties editor](/blog/img/2014-11-23/default-properties-editor.jpg "Default properties editor")

Но, у шапки таблицы стиль обычно отличен от стиля остальных строк, 
например первая строка может быть написана болдом или иметь цветовую заливку. 
Тут в дело вступает **Редактор таблицы**, он позволяет настроить каждую отдельную строку, колонку или ячейку по своему. 
Кроме того стандартный текст, для той же шапки таблицы, то же указывается в **Редактор таблицы**.

![Special properties editor](/blog/img/2014-11-23/special-properties-editor.jpg "Special properties editor")

Получается чтобы полностью скопировать стиль таблицы нужно копировать два набора параметров.

{% highlight c# %}
public static IInPictureGridFigure ClonaTable(IInPictureGridFigure target, IInPictureGridFigure source)
{
	target.ColCount = source.ColCount;
	target.RowCount = source.RowCount;
	target.Width = source.Width;
	target.Height = source.Height;


	var targetGridFormat = target.GridFormat;
	var sourceGridFormat = source.GridFormat;

	if (sourceGridFormat.ContainsKinds(TInGridFormatKind.ingfLeftPen))
	{
		targetGridFormat = ClonePen(TInGridFormatKind.ingfLeftPen, targetGridFormat, sourceGridFormat);
	}

	if (sourceGridFormat.ContainsKinds(TInGridFormatKind.ingfRightPen))
	{
		targetGridFormat = ClonePen(TInGridFormatKind.ingfRightPen, targetGridFormat, sourceGridFormat);
	}

	if (sourceGridFormat.ContainsKinds(TInGridFormatKind.ingfTopPen))
	{
		targetGridFormat = ClonePen(TInGridFormatKind.ingfTopPen, targetGridFormat, sourceGridFormat);
	}

	if (sourceGridFormat.ContainsKinds(TInGridFormatKind.ingfBottomPen))
	{
		targetGridFormat = ClonePen(TInGridFormatKind.ingfBottomPen, targetGridFormat, sourceGridFormat);
	}
	targetGridFormat.Update();

	for (int i = 0; i < source.ColCount; i++)
	{
		var targetColumnFormat = target.GetFormat(i, -1);
		var sourceColumnFormat = source.GetFormat(i, -1);

		var defaultSourceColumnFormat = source.ColFormat;
		var defaultTargetColumnFormat = target.ColFormat;

		if (sourceColumnFormat.ContainsKinds(TInGridFormatKind.ingfColWidth))
		{
			targetColumnFormat = CloneSize(TInGridFormatKind.ingfColWidth, targetColumnFormat, sourceColumnFormat);
		}
		else
		{
			defaultTargetColumnFormat = CloneSize(TInGridFormatKind.ingfColWidth, defaultTargetColumnFormat, defaultSourceColumnFormat);
		}

		if (sourceColumnFormat.ContainsKinds(TInGridFormatKind.ingfFont))
		{
			targetColumnFormat = CloneFont(TInGridFormatKind.ingfFont, targetColumnFormat, sourceColumnFormat);
		}

		if (sourceColumnFormat.ContainsKinds(TInGridFormatKind.ingfTextFormat))
		{
			targetColumnFormat = CloneTextFormat(TInGridFormatKind.ingfFont, targetColumnFormat, sourceColumnFormat);
		}

		if (sourceColumnFormat.ContainsKinds(TInGridFormatKind.ingfLeftPen))
		{
			targetColumnFormat = ClonePen(TInGridFormatKind.ingfLeftPen, targetColumnFormat, sourceColumnFormat);
		}
		else
		{
			defaultTargetColumnFormat = ClonePen(TInGridFormatKind.ingfLeftPen, defaultTargetColumnFormat, defaultSourceColumnFormat);
		}

		if (sourceColumnFormat.ContainsKinds(TInGridFormatKind.ingfRightPen))
		{
			targetColumnFormat = ClonePen(TInGridFormatKind.ingfRightPen, targetColumnFormat, sourceColumnFormat);
		}
		else
		{
			defaultTargetColumnFormat = ClonePen(TInGridFormatKind.ingfRightPen, defaultTargetColumnFormat, defaultSourceColumnFormat);
		}

		if (sourceColumnFormat.ContainsKinds(TInGridFormatKind.ingfTopPen))
		{
			targetColumnFormat = ClonePen(TInGridFormatKind.ingfTopPen, targetColumnFormat, sourceColumnFormat);
		}
		else
		{
			defaultTargetColumnFormat = ClonePen(TInGridFormatKind.ingfTopPen, defaultTargetColumnFormat, defaultSourceColumnFormat);
		}

		if (sourceColumnFormat.ContainsKinds(TInGridFormatKind.ingfBottomPen))
		{
			targetColumnFormat = ClonePen(TInGridFormatKind.ingfBottomPen, targetColumnFormat, sourceColumnFormat);
		}
		else
		{
			defaultTargetColumnFormat = ClonePen(TInGridFormatKind.ingfBottomPen, defaultTargetColumnFormat, defaultSourceColumnFormat);
		}

		defaultTargetColumnFormat.Update();
		targetColumnFormat.Update();
	}

	for (int i = 0; i < source.RowCount; i++)
	{
		var targetRowFormat = target.GetFormat(-1, i);
		var sourceRowFormat = source.GetFormat(-1, i);

		var defaultSourceRowFormat = source.RowFormat;
		var defaultTargetRowFormat = target.RowFormat;

		if (sourceRowFormat.ContainsKinds(TInGridFormatKind.ingfRowHeight))
		{
			targetRowFormat = CloneSize(TInGridFormatKind.ingfColWidth, targetRowFormat, sourceRowFormat);
		}
		else
		{
			defaultTargetRowFormat = CloneSize(TInGridFormatKind.ingfColWidth, defaultTargetRowFormat, defaultSourceRowFormat);
		}


		if (sourceRowFormat.ContainsKinds(TInGridFormatKind.ingfFont))
		{
			targetRowFormat = CloneFont(TInGridFormatKind.ingfFont, targetRowFormat, sourceRowFormat);
		}

		if (sourceRowFormat.ContainsKinds(TInGridFormatKind.ingfTextFormat))
		{
			targetRowFormat = CloneTextFormat(TInGridFormatKind.ingfFont, targetRowFormat, sourceRowFormat);
		}

		if (sourceRowFormat.ContainsKinds(TInGridFormatKind.ingfLeftPen))
		{
			targetRowFormat = ClonePen(TInGridFormatKind.ingfLeftPen, targetRowFormat, sourceRowFormat);
		}
		else
		{
			defaultTargetRowFormat = ClonePen(TInGridFormatKind.ingfLeftPen, defaultTargetRowFormat, defaultSourceRowFormat);
		}

		if (sourceRowFormat.ContainsKinds(TInGridFormatKind.ingfRightPen))
		{
			targetRowFormat = ClonePen(TInGridFormatKind.ingfRightPen, targetRowFormat, sourceRowFormat);
		}
		else
		{
			defaultTargetRowFormat = ClonePen(TInGridFormatKind.ingfRightPen, defaultTargetRowFormat, defaultSourceRowFormat);
		}

		if (sourceRowFormat.ContainsKinds(TInGridFormatKind.ingfTopPen))
		{
			targetRowFormat = ClonePen(TInGridFormatKind.ingfTopPen, targetRowFormat, sourceRowFormat);
		}
		else
		{
			defaultTargetRowFormat = ClonePen(TInGridFormatKind.ingfTopPen, defaultTargetRowFormat, defaultSourceRowFormat);
		}

		if (sourceRowFormat.ContainsKinds(TInGridFormatKind.ingfBottomPen))
		{
			targetRowFormat = ClonePen(TInGridFormatKind.ingfBottomPen, targetRowFormat, sourceRowFormat);
		}
		else
		{
			defaultTargetRowFormat = ClonePen(TInGridFormatKind.ingfBottomPen, defaultTargetRowFormat, defaultSourceRowFormat);
		}

		targetRowFormat.Update();
		defaultTargetRowFormat.Update();
	}

	for (int i = 0; i < source.ColCount; i++)
	{
		for (int j = 0; j < source.RowCount; j++)
		{
			var targetCellFormat = target.GetFormat(i, j);
			var sourceCellFormat = source.GetFormat(i, j);

			var defaultSourceCellFormat = source.CellFormat;
			var defaultTargetCellFormat = target.CellFormat;

			if (sourceCellFormat.ContainsKinds(TInGridFormatKind.ingfFont))
			{
				targetCellFormat = CloneFont(TInGridFormatKind.ingfFont, targetCellFormat, sourceCellFormat);
			}
			else
			{
				defaultTargetCellFormat = CloneFont(TInGridFormatKind.ingfFont, defaultTargetCellFormat, defaultSourceCellFormat);
			}

			if (sourceCellFormat.ContainsKinds(TInGridFormatKind.ingfTextFormat))
			{
				targetCellFormat = CloneTextFormat(TInGridFormatKind.ingfFont, targetCellFormat, sourceCellFormat);
			}
			else
			{
				defaultTargetCellFormat = CloneTextFormat(TInGridFormatKind.ingfFont, defaultTargetCellFormat, defaultSourceCellFormat);
			}

			if (sourceCellFormat.ContainsKinds(TInGridFormatKind.ingfBrush))
			{
				targetCellFormat = CloneBrush(TInGridFormatKind.ingfFont, targetCellFormat, sourceCellFormat);
			}
			else
			{
				defaultTargetCellFormat = CloneBrush(TInGridFormatKind.ingfFont, defaultTargetCellFormat, defaultSourceCellFormat);
			}

			if (sourceCellFormat.ContainsKinds(TInGridFormatKind.ingfLeftIndent))
			{
				targetCellFormat = CloneSize(TInGridFormatKind.ingfColWidth, targetCellFormat, sourceCellFormat);
			}
			else
			{
				defaultTargetCellFormat = CloneSize(TInGridFormatKind.ingfColWidth, defaultTargetCellFormat, defaultSourceCellFormat);
			}

			targetCellFormat.Update();
			defaultTargetCellFormat.Update();
		}
	}

	for (int i = 0; i < source.ColCount; i++)
	{
		for (int j = 0; j < source.RowCount; j++)
		{
			target.Text[i, j] = source.Text[i, j];
		}
	}

	return target;
}
{% endhighlight %}

Метод состоит из трех циклов по строкам **(RowCount)**, столбцам **(ColCount)**, и ячейкам. 
Для каждого элемента определяется есть ли у него специальный формат с помощью метода **ContainsKinds()**. 
Если специальный формат есть, то он копируется в новую таблицу, если нет то копируется дефолтный формат. 
Для удобства код копирования отдельных элементов формата вынесен в отдельные методы:

{% highlight c# %}
private static IInGridFormat ClonePen(TInGridFormatKind inGridFormatKind, IInGridFormat target, IInGridFormat source)
{
	target.Pen[inGridFormatKind].Style = source.Pen[inGridFormatKind].Style;
	target.Pen[inGridFormatKind].Color = source.Pen[inGridFormatKind].Color;
	target.Pen[inGridFormatKind].Mode = source.Pen[inGridFormatKind].Mode;
	target.Pen[inGridFormatKind].WidthInMM = source.Pen[inGridFormatKind].WidthInMM;

	return target;
}
{% endhighlight %}

{% highlight c# %}
private static IInGridFormat CloneFont(TInGridFormatKind inGridFormatKind, IInGridFormat target, IInGridFormat source)
{
	target.Font[inGridFormatKind].Size = source.Font[inGridFormatKind].Size;
	target.Font[inGridFormatKind].Style = source.Font[inGridFormatKind].Style;

	return target;
}
{% endhighlight %}

{% highlight c# %}
private static IInGridFormat CloneBrush(TInGridFormatKind inGridFormatKind, IInGridFormat target, IInGridFormat source)
{
	target.Brush[TInGridFormatKind.ingfBrush].BackColor = source.Brush[TInGridFormatKind.ingfBrush].BackColor;
	target.Brush[TInGridFormatKind.ingfBrush].HatchColor = source.Brush[TInGridFormatKind.ingfBrush].HatchColor;
	target.Brush[TInGridFormatKind.ingfBrush].Transparency = source.Brush[TInGridFormatKind.ingfBrush].Transparency;

	return target;
}
{% endhighlight %}

{% highlight c# %}
private static IInGridFormat CloneSize(TInGridFormatKind inGridFormatKind, IInGridFormat target,
                                                      IInGridFormat source)
{
	target.Size[inGridFormatKind] = source.Size[inGridFormatKind];

	return target;
}

{% endhighlight %}

{% highlight c# %}
private static IInGridFormat CloneTextFormat(TInGridFormatKind inGridFormatKind, IInGridFormat target,
                                              IInGridFormat source)
{
	target.TextFormat[inGridFormatKind] = source.TextFormat[inGridFormatKind];

	return target;
}
{% endhighlight %}

В заключение скажу, что методы копирования не являются полными, я копировал только те элементы формата, которые использовались в моем макете печати.