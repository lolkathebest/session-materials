# Работа с файлами в оконном приложении

*Основы алгоритмизации и программирования (ОП.04) — C#* · тема 38

> Конспект посвящен интеграции файловой системы в приложения Windows Forms на платформе .NET 8 с использованием C#. Рассматриваются встроенные диалоговые окна (OpenFileDialog, SaveFileDialog, FolderBrowserDialog), потоки ввода-вывода (StreamReader, StreamWriter, FileStream, BinaryReader, BinaryWriter), а также обработка исключений при работе с файлами и сохранение пользовательских настроек.

**После изучения вы сможете:**

- Применять диалоговые окна OpenFileDialog и SaveFileDialog для выбора путей к файлам в интерфейсе Windows Forms
- Реализовывать сохранение и чтение текстовых данных из элементов управления TextBox, ListBox и DataGridView
- Проектировать сериализацию и десериализацию пользовательских структур данных в бинарном и текстовом форматах
- Обрабатывать ошибки ввода-вывода (IOException, UnauthorizedAccessException) с помощью блоков try-catch
- Организовывать автоматическое сохранение и загрузку состояния формы при запуске и закрытии приложения

**Что нужно помнить из прошлых тем:**

- Понятие потоков ввода-вывода (StreamReader, StreamWriter, BinaryReader, BinaryWriter) из темы 13
- Обработка событий формы (FormLoad, FormClosing, Click) из темы 33
- Работа с элементами управления TextBox, ListBox, MenuStrip и диалоговыми окнами из темы 34
- Создание и использование пользовательских структур (struct) и классов для представления данных

## Содержание

1. 1. Стандартные диалоговые окна файловой системы в Windows Forms
2. 2. Чтение и запись текстовых файлов из элементов интерфейса
3. 3. Бинарные файлы и табличные данные в DataGridView
4. 4. Обработка исключений и обеспечение отказоустойчивости при файловых операциях
5. 5. Автоматическое сохранение настроек приложения и сессий

## 1. Стандартные диалоговые окна файловой системы в Windows Forms

### Назначение и архитектура компонентов OpenFileDialog, SaveFileDialog и FolderBrowserDialog

При разработке настольных приложений на платформе .NET 8 с использованием библиотеки Windows Forms возникает необходимость взаимодействия с файловой системой операционной системы. Пользователь должен иметь возможность указать, откуда прочитать данные или куда сохранить результаты работы программы. 

Использование жестко закодированных путей к файлам (например, `"C:\\data\\file.txt"`) является грубой ошибкой проектирования, так как на компьютере пользователя соответствующая папка может отсутствовать или у приложения не будет прав на запись в этот каталог. Для решения этой задачи применяются стандартные диалоговые окна операционной системы. Преимущество использования стандартных диалогов заключается в том, что пользователь видит привычный интерфейс проводника Windows, к которому он адаптирован в повседневной работе с операционной системой.

В пространстве имен `System.Windows.Forms` предусмотрены следующие классы для реализации диалогов:

1. `OpenFileDialog` — компонент, предназначенный для выбора одного или нескольких существующих файлов для чтения.
2. `SaveFileDialog` — компонент, предназначенный для указания пути и имени файла, в который приложение будет записывать данные. Если файл с указанным именем уже существует, этот диалог стандартными средствами может предупредить пользователя о перезаписи.
3. `FolderBrowserDialog` — компонент, используемый для выбора папки (директории), а не конкретного файла.

Архитектурно все эти компоненты унаследованы от базового абстрактного класса `CommonDialog`, который в свою очередь происходит от класса `Component`. Это означает, что данные компоненты могут быть добавлены как через визуальный дизайнер Visual Studio (путем перетаскивания с панели элементов), так и созданы программно с помощью оператора `new`. 

> **Определение.** Компонент диалогового окна в Windows Forms — это объектно-ориентированная обертка над системными функциями API операционной системы (Common Item Dialog), которая инкапсулирует логику отрисовки окон проводника и возвращает результат выбора пользователя в управляемый код C#.

Пример создания компонента `OpenFileDialog` в коде:

```csharp
OpenFileDialog openFileDialog = new OpenFileDialog();
```

Принципиальное отличие программного создания от размещения на форме через визуальный дизайнер заключается в том, что созданный через `new` объект существует только в рамках метода и удаляется сборщиком мусора после завершения работы с ним, тогда как компонент с панели элементов является управляемым полем класса формы.

---

### Основные свойства диалоговых окон: Filter, Title, InitialDirectory, DefaultExt, RestoreDirectory

Для настройки внешнего вида и поведения диалоговых окон используются специальные свойства. Рассмотрим каждое из них подробно:

- `Title` (тип `string`) — устанавливает заголовок (текст в шапке) диалогового окна. По умолчанию используется стандартный системный заголовок («Открыть» или «Сохранить как»), но его рекомендуется изменять для уточнения контекста задачи (например, «Выберите файл отчета за текущий месяц»).
- `InitialDirectory` (тип `string`) — задает путь к каталогу, который будет открыт при первом появлении диалогового окна. Если свойство не задано или содержит пустую строку, система открывает папку, которая использовалась в последний раз для этого диалога, либо системную папку документов по умолчанию.
- `DefaultExt` (тип `string`) — расширение файла по умолчанию (без точки, например, `"txt"`). Если пользователь вводит имя файла без указания расширения, это расширение добавляется автоматически (актуально преимущественно для `SaveFileDialog`).
- `RestoreDirectory` (тип `bool`) — логическое свойство. Если его значение равно `true`, то диалоговое окно при закрытии восстанавливает текущую рабочую директорию приложения в то состояние, которое было до открытия диалога. Рекомендуется устанавливать в `true` во избежание непреднамеренного изменения рабочей папки приложения.
- `Filter` (тип `string`) — строка, определяющая фильтрацию файлов по их расширениям в выпадающем списке типа файлов. Это критически важное свойство, защищающий пользователя от выбора файлов неподходящего формата.

Синтаксис строки фильтра строго регламентирован. Строка состоит из пар описаний и масок, разделенных вертикальной чертой (`|`). Общий формат записи выглядит следующим образом:

`«Описание для пользователя (маска)|маска|Описание для пользователя 2 (маска2)|маска2»`

Например, строка фильтра для текстовых файлов и всех остальных файлов записывается так:

`"Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*"`

В этой строке первая часть (`Текстовые файлы (*.txt)`) отображается в выпадающем списке диалогового окна, а вторая часть (`*.txt`) указывает маску отбора. Если необходимо указать несколько расширений для одной маски, они разделяются точкой с запятой: `*.txt;*.log`.

---

### Метод ShowDialog() и возвращаемое значение DialogResult (OK, Cancel)

Диалоговые окна в Windows Forms являются модальными. Это означает, что пока диалоговое окно открыто, пользователь не может переключиться на главное окно приложения и продолжить работу с ним, пока не закроет или не отменит диалог.

Для отображения диалогового окна используется метод `ShowDialog()`. Данный метод останавливает выполнение текущего метода в коде до тех пор, пока пользователь не выберет файл и не нажмет кнопку подтверждения, либо не закроет окно (не нажмет «Отмена»).

Метод `ShowDialog()` возвращает значение перечисления `DialogResult`. Данное перечисление содержит все возможные варианты завершения работы с диалоговым окном. Основные константы перечисления `DialogResult`:

- `DialogResult.OK` — пользователь нажал кнопку подтверждения выбора («Открыть» или «Сохранить»).
- `DialogResult.Cancel` — пользователь нажал кнопку «Отмена», закрыл окно крестиком или нажал клавишу `Esc`.

Проверка результата выполнения диалога обязательна в любой программе, так как обращение к выбранному пути при отмене операции приведет к логическим ошибкам. Типичная структура кода для вызова диалога выглядит следующим образом:

```csharp
OpenFileDialog openFileDialog = new OpenFileDialog();
openFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt";

// Вызываем окно и проверяем, нажал ли пользователь кнопку OK
if (openFileDialog.ShowDialog() == DialogResult.OK)
{
    // Пользователь выбрал файл и подтвердил выбор. 
    // Здесь происходит работа с файлом.
    string filePath = openFileDialog.FileName;
}
else
{
    // Пользователь нажал «Отмена». Действие отменяется.
}
```

---

### Получение выбранного пути через свойство FileName и FileNames (множественный выбор)

После того как пользователь выбрал файл и диалоговое окно закрылось с результатом `DialogResult.OK`, разработчику необходимо получить путь к выбранному файлу для его дальнейшей обработки.

Для этого в классах `OpenFileDialog` и `SaveFileDialog` предусмотрены свойства:

- `FileName` (тип `string`) — возвращает полный путь к первому (или единственному) выбранному файлу. Если файл не выбран, свойство возвращает пустую строку `""`.
- `FileNames` (тип массива строк `string[]`) — возвращает массив путей ко всем выбранным файлам. Это свойство используется, когда разрешен множественный выбор файлов.

Для включения режима множественного выбора в компоненте `OpenFileDialog` необходимо установить логическое свойство `Multiselect` в значение `true`:

```csharp
OpenFileDialog openFileDialog = new OpenFileDialog();
openFileDialog.Multiselect = true; // Разрешаем выбор нескольких файлов
openFileDialog.Filter = "Все файлы (*.*)|*.*";

if (openFileDialog.ShowDialog() == DialogResult.OK)
{
    // Проходим по всем выбранным путям в цикле
    foreach (string filePath in openFileDialog.FileNames)
    {
        // Обработка каждого отдельного файла
    }
}
```

---

### Фильтрация файлов по расширениям: синтаксис строки фильтра

Рассмотрим подробнее механизм работы свойства `Filter`. Ошибки в синтаксисе строки фильтра не вызывают исключений времени компиляции, однако приводят к тому, что выпадающий список в диалоговом окне отображается некорректно или файлы не фильтруются.

Строка фильтра обрабатывается парсером операционной системы попарно. Каждая пара состоит из отображаемого текста и шаблона поиска. Между собой пары также разделяются символом вертикальной черты (`|`).

Пример сложного составного фильтра для программы обработки изображений:

```csharp
saveFileDialog1.Filter = "Графические файлы (*.png)|*.png|Файлы JPEG (*.jpg;*.jpeg)|*.jpg;*.jpeg|Все файлы (*.*)|*.*";
```

Если в строке фильтра пропустить вертикальную черту или поставить лишний разделитель, система выбросит исключение `ArgumentException` при попытке отобразить окно `ShowDialog()`.

Свойство `FilterIndex` позволяет указать, какой именно пункт из списка фильтров будет выбран по умолчанию при открытии окна (нумерация индексов начинается с 1). Например, если задать `saveFileDialog1.FilterIndex = 2`, то по умолчанию в списке будет выбран второй пункт фильтра.

---

### Обеспечение безопасности при работе с путями в среде .NET 8

При разработке приложений на платформе .NET 8 вопросы безопасности работы с файловой системой стоят на первом месте. Современные операционные системы (Windows 10/11) ограничивают права приложений на запись в защищенные системные директории (например, `C:\Windows`, `C:\Program Files`).

При проектировании интерфейса с использованием `OpenFileDialog` и `SaveFileDialog` следует учитывать следующие аспекты безопасности:

1. **Проверка существования файла при открытии:** Компонент `OpenFileDialog` по умолчанию имеет свойство `CheckFileExists = true`. Это гарантирует, что пользователь не сможет выбрать несуществующий файл (в отличие от `SaveFileDialog`, где файл может создаваться заново). Если пользователь введет вручную несуществующий путь, система сама покажет предупреждение.
2. **Предупреждение о перезаписи:** Компонент `SaveFileDialog` по умолчанию имеет свойство `CheckPathExists = true` и `OverwritePrompt = true`. Свойство `OverwritePrompt` заставляет систему автоматически выводить стандартное окно с вопросом: «Файл уже существует. Заменить его?», защищая пользователя от случайной потери данных.
3. **Очистка и проверка путей:** Полученные из свойства `FileName` строки могут содержать недопустимые символы или быть подвержены атакам типа Path Traversal, если путь формируется на основе внешних данных. При работе с диалоговыми окнами эта проблема минимизируется, так как диалог возвращает абсолютно корректный абсолютный путь, проверенный операционной системой. Тем не менее, перед открытием потока рекомендуется использовать статические методы класса `Path` (`Path.GetFullPath`, `Path.GetDirectoryName`), чтобы убедиться в корректности структуры пути.

---

### Пример 1. Реализация меню 'Открыть' и 'Сохранить как' с настройкой фильтра для текстовых файлов (.txt) и всех файлов (*.*)

Рассмотрим разработку фрагмента оконного приложения (класс формы `MainForm`), в котором реализовано главное меню (`MenuStrip`) с пунктами «Файл -> Открыть» и «Файл -> Сохранить как». Текст отображается в многострочном поле ввода `TextBox` с именем `textBoxContent`.

#### Рисунок словами (схема интерфейса)
```text
+-------------------------------------------------------+
| Файл  Справка                                         |
+-------------------------------------------------------+
| [ Открыть...   ]                                      |
| [ Сохранить как]                                      |
+-------------------------------------------------------+
|                                                       |
|  Здесь отображается текст открытого файла...          |
|  (TextBox, Multiline = true, Dock = Fill)             |
|                                                       |
+-------------------------------------------------------+
```

#### Ход рассуждений
1. Для реализации меню используем компонент `MenuStrip`, в котором создаем пункты меню. Для каждого пункта создается обработчик события клика (`Click`).
2. При нажатии на пункт «Открыть» создаем экземпляр `OpenFileDialog`. Настраиваем его фильтр так, чтобы пользователь мог выбрать либо текстовые файлы, либо любые другие.
3. Проверяем, нажал ли пользователь кнопку `OK` в диалоговом окне. Если да, получаем путь к файлу через свойство `FileName`.
4. Для чтения текстового файла используем класс `StreamReader` (внутри конструкции `using`, чтобы гарантировать закрытие файла и освобождение системных ресурсов). Загружаем весь текст в свойство `Text` элемента `textBoxContent`.
5. При нажатии на пункт «Сохранить как» создаем экземпляр `SaveFileDialog`. Настраиваем аналогичный фильтр и дефолтное расширение `.txt`.
6. Если пользователь подтверждает выбор пути, записываем содержимое `textBoxContent.Text` в выбранный файл с помощью класса `StreamWriter`.

#### Полный исходный код решения

```csharp
using System;
using System.IO;
using System.Windows.Forms;
using System.Text;

namespace FileDialogApp
{
    public partial class MainForm : Form
    {
        private TextBox textBoxContent;
        private MenuStrip menuStripMain;
        private ToolStripMenuItem menuItemFile;
        private ToolStripMenuItem menuItemOpen;
        private ToolStripMenuItem menuItemSaveAs;

        public MainForm()
        {
            InitializeComponentCustom();
        }

        private void InitializeComponentCustom()
        {
            // Настройка формы
            this.Text = "Текстовый редактор с диалогами";
            this.Width = 600;
            this.Height = 400;

            // Создание текстового поля
            this.textBoxContent = new TextBox();
            this.textBoxContent.Multiline = true;
            this.textBoxContent.Dock = DockStyle.Fill;
            this.textBoxContent.ScrollBars = ScrollBars.Vertical;

            // Создание меню
            this.menuStripMain = new MenuStrip();
            this.menuItemFile = new ToolStripMenuItem("Файл");
            
            this.menuItemOpen = new ToolStripMenuItem("Открыть...");
            this.menuItemOpen.Click += new EventHandler(this.MenuItemOpen_Click);

            this.menuItemSaveAs = new ToolStripMenuItem("Сохранить как...");
            this.menuItemSaveAs.Click += new EventHandler(this.MenuItemSaveAs_Click);

            // Сборка меню
            this.menuItemFile.DropDownItems.Add(this.menuItemOpen);
            this.menuItemFile.DropDownItems.Add(this.menuItemSaveAs);
            this.menuStripMain.Items.Add(this.menuItemFile);

            // Добавление элементов на форму
            this.Controls.Add(this.textBoxContent);
            this.Controls.Add(this.menuStripMain);
            this.MainMenuStrip = this.menuStripMain;
        }

        // Обработчик пункта меню «Открыть»
        private void MenuItemOpen_Click(object sender, EventArgs e)
        {
            using (OpenFileDialog openFileDialog = new OpenFileDialog())
            {
                openFileDialog.Title = "Выберите текстовый документ";
                openFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
                openFileDialog.FilterIndex = 1;
                openFileDialog.RestoreDirectory = true;

                if (openFileDialog.ShowDialog() == DialogResult.OK)
                {
                    string filePath = openFileDialog.FileName;

                    try
                    {
                        // Чтение файла с использованием кодировки UTF-8
                        using (StreamReader reader = new StreamReader(filePath, Encoding.UTF8))
                        {
                            this.textBoxContent.Text = reader.ReadToEnd();
                        }
                        MessageBox.Show("Файл успешно открыт!", "Информация", MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    catch (IOException ex)
                    {
                        MessageBox.Show($"Ошибка при чтении файла: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
        }

        // Обработчик пункта меню «Сохранить как»
        private void MenuItemSaveAs_Click(object sender, EventArgs e)
        {
            using (SaveFileDialog saveFileDialog = new SaveFileDialog())
            {
                saveFileDialog.Title = "Сохранить документ как";
                saveFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
                saveFileDialog.DefaultExt = "txt";
                saveFileDialog.AddExtension = true;
                saveFileDialog.RestoreDirectory = true;

                if (saveFileDialog.ShowDialog() == DialogResult.OK)
                {
                    string filePath = saveFileDialog.FileName;

                    try
                    {
                        // Запись файла с использованием кодировки UTF-8
                        using (StreamWriter writer = new StreamWriter(filePath, false, Encoding.UTF8))
                        {
                            writer.Write(this.textBoxContent.Text);
                        }
                        MessageBox.Show("Файл успешно сохранен!", "Информация", MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    catch (IOException ex)
                    {
                        MessageBox.Show($"Ошибка при записи файла: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
        }
    }
}
```

#### Проверка и анализ
- **Единицы и типы:** Пути обрабатываются как строковые переменные (`string`), кодировка явно задана через `Encoding.UTF8`, что исключает проблемы с отображением кириллицы при переносе файлов между различными операционными системами.
- **Правдоподобность:** Использование блоков `using` гарантирует освобождение дескрипторов файлов сразу после окончания операций чтения и записи. Проверка `DialogResult.OK` предотвращает возникновение исключений при отмене выбора в окне.

---

### Пример 2. Создание диалога сохранения отчета с проверкой существования файла и подтверждением перезаписи

В корпоративных приложениях часто требуется формировать отчеты и сохранять их на жесткий диск. Если пользователь случайно указывает имя уже существующего отчета, необходимо явным образом подтвердить намерение перезаписать данные или предложить изменить имя файла.

Компонент `SaveFileDialog` по умолчанию поддерживает свойство `OverwritePrompt = true`, однако в ряде случаев разработчику требуется реализовать кастомную логику подтверждения с использованием дополнительных проверок и выводом специализированных сообщений через `MessageBox`.

#### Рисунок словами (схема алгоритма)
```text
[ Нажатие кнопки 'Создать отчет' ]
               │
               ▼
   [ Вызов SaveFileDialog ] ──(Отмена)──> [ Выход ]
               │
         (Нажато OK)
               │
               ▼
   [ Существует ли файл? ] ──(Да)──> [ Запрос MessageBox: Перезаписать? ]
               │                          │
             (Нет)                 (Нажато No) ──> [ Возврат к диалогу ]
               │                          │
               └───────────┬──────────────┘
                           │ (Нажато Yes / Файла нет)
                           ▼
               [ Генерация и запись отчета ]
                           │
                           ▼
               [ Сообщение об успехе ]
```

#### Ход рассуждений
1. На форме размещается кнопка `buttonCreateReport`. По нажатию открывается `SaveFileDialog`.
2. Настраиваем свойства диалога: заголовок «Сохранить бухгалтерский отчет», фильтр для отчетов с расширением `.rep` и `.txt`.
3. После получения `DialogResult.OK` мы программно проверяем существование файла с помощью статического метода `File.Exists(filePath)`.
4. Если файл существует, выводим модальное окно `MessageBox.Show` с кнопками `YesNo` и предупреждающим знаком (`MessageBoxIcon.Warning`). Если пользователь отвечает `DialogResult.No`, прерываем операцию сохранения и возвращаем управление пользователю.
5. Выполняем имитацию генерации отчета и записываем данные на диск.

#### Полный исходный код решения

```csharp
using System;
using System.IO;
using System.Windows.Forms;
using System.Text;

namespace ReportSaverApp
{
    public partial class ReportForm : Form
    {
        private Button buttonCreateReport;
        private TextBox textBoxReportData;

        public ReportForm()
        {
            InitializeComponentCustom();
        }

        private void InitializeComponentCustom()
        {
            this.Text = "Генератор отчетов";
            this.Width = 500;
            this.Height = 350;

            // Текстовое поле с данными отчета
            this.textBoxReportData = new TextBox();
            this.textBoxReportData.Multiline = true;
            this.textBoxReportData.Location = new System.Drawing.Point(12, 12);
            this.textBoxReportData.Size = new System.Drawing.Size(460, 230);
            this.textBoxReportData.Text = "ОТЧЕТ О ПРОДАЖАХ\r\nПериод: Сентябрь\r\nСумма: 154000 руб.\r\nСтатус: Успешно";

            // Кнопка сохранения отчета
            this.buttonCreateReport = new Button();
            this.buttonCreateReport.Text = "Сохранить отчет...";
            this.buttonCreateReport.Location = new System.Drawing.Point(12, 260);
            this.buttonCreateReport.Size = new System.Drawing.Size(150, 30);
            this.buttonCreateReport.Click += new EventHandler(this.ButtonCreateReport_Click);

            this.Controls.Add(this.textBoxReportData);
            this.Controls.Add(this.buttonCreateReport);
        }

        private void ButtonCreateReport_Click(object sender, EventArgs e)
        {
            using (SaveFileDialog saveDialog = new SaveFileDialog())
            {
                saveDialog.Title = "Сохранить бухгалтерский отчет";
                saveDialog.Filter = "Файлы отчетов (*.rep)|*.rep|Текстовые файлы (*.txt)|*.txt";
                saveDialog.DefaultExt = "rep";
                saveDialog.FileName = "Report_Sept";

                // Показываем диалог
                if (saveDialog.ShowDialog() == DialogResult.OK)
                {
                    string path = saveDialog.FileName;

                    // Дополнительная проверка существования файла (на случай ручного ввода или отключенных системных флагов)
                    if (File.Exists(path))
                    {
                        DialogResult overwriteResult = MessageBox.Show(
                            "Файл с таким именем уже существует на диске.\nВы хотите заменить его?",
                            "Подтверждение перезаписи",
                            MessageBoxButtons.YesNo,
                            MessageBoxIcon.Warning);

                        if (overwriteResult == DialogResult.No)
                        {
                            // Пользователь отказался перезаписывать файл, прерываем процедуру сохранения
                            return;
                        }
                    }

                    try
                    {
                        // Запись текста отчета в файл
                        using (StreamWriter writer = new StreamWriter(path, false, Encoding.UTF8))
                        {
                            writer.Write(this.textBoxReportData.Text);
                        }

                        MessageBox.Show(
                            "Отчет успешно сохранен в файл:\n" + path,
                            "Успех",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Information);
                    }
                    catspac: // метка не нужна, используем стандартный блок catch
                    catch (UnauthorizedAccessException)
                    {
                        MessageBox.Show(
                            "Отказано в доступе. У вас нет прав на запись в выбранную директорию.",
                            "Ошибка безопасности",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Error);
                    }
                    catch (IOException ex)
                    {
                        MessageBox.Show(
                            $"Произошла ошибка ввода-вывода: {ex.Message}",
                            "Ошибка",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Error);
                    }
                }
            }
        }
    }
}
```

#### Проверка и анализ
- **Устойчивость к сбоям:** Код содержит перехват специализированных исключений (`UnauthorizedAccessException` и `IOException`), что позволяет программе не завершаться аварийно при попытке записи в защищенные папки или при извлечении флеш-накопителя в процессе сохранения.
- **Интерфейс:** Диалоговые окна выдают информативные сообщения, полностью соответствующие требованиям отказоустойчивости настольных приложений.

Ответ: Раздел полностью раскрывает назначение, архитектуру, свойства и программное использование стандартных диалоговых окон файловой системы (`OpenFileDialog`, `SaveFileDialog`) в средах Windows Forms на базе .NET 8 с примерами кода и строгим соблюдением стандартов безопасности.

## 2. Чтение и запись текстовых файлов из элементов интерфейса

### Связывание многострочного TextBox с текстовыми потоками

Интерфейс Windows Forms предоставляет элемент управления `TextBox`, который в однострочном режиме служит для ввода коротких строк, а при установке свойства `Multiline = true` превращается в полноценное многострочное поле для работы с текстом. Для того чтобы связать это поле с файлом на жестком диске, используются классы `StreamReader` (для чтения) и `StreamWriter` (для записи) из пространства имен `System.IO`.

> **Определение.** `StreamReader` — это класс, предназначенный для чтения символов из байтового потока с учетом определенной кодировки. `StreamWriter` — симметричный класс для записи символов в поток.

Перед тем как использовать потоки, необходимо понять, как текст из `TextBox` попадает в файл и наоборот. В памяти `TextBox` хранит текст как единую строку, в которой строки разделены специальными символами перевода строки. В операционной системе Windows стандартным разделителем является пара символов `\r\n` (возврат каретки `CR` и перевод строки `LF`), в то время как в Linux и macOS используется только `LF` (`\n`). 

Чтобы приложение было кроссплатформенным или корректно обрабатывало файлы, созданные в разных системах, используется статическое свойство `Environment.NewLine`. Оно возвращает актуальный для текущей операционной системы символ (или последовательность символов) перевода строки.

Чтение всего содержимого файла в многострочный `TextBox` выполняется одной строкой с помощью метода `File.ReadAllText` или потока `StreamReader`:

```csharp
// Чтение с использованием StreamReader
using (StreamReader reader = new StreamReader("document.txt", Encoding.UTF8))
{
    textBox1.Text = reader.ReadToEnd();
}
```

Запись содержимого `TextBox` обратно в файл происходит зеркально:

```csharp
// Запись с использованием StreamWriter
using (StreamWriter writer = new StreamWriter("document.txt", false, Encoding.UTF8))
{
    writer.Write(textBox1.Text);
}
```

### Перенос данных между ListBox, ComboBox и текстовыми файлами

Элементы управления `ListBox` и `ComboBox` содержат коллекции элементов (`Items`), которые во время работы программы представляют собой набор объектов (чаще всего строк). Для сохранения такого списка в файл каждый элемент коллекции записывается на отдельной строке.

Для записи всех элементов `ListBox` в текстовый файл используется цикл `foreach`, обходящий коллекцию `Items`:

```csharp
using (StreamWriter writer = new StreamWriter("tasks.txt", false, Encoding.UTF8))
{
    foreach (var item in listBox1.Items)
    {
        // Превращаем элемент в строку и записываем с переводом строки
        writer.WriteLine(item.ToString());
    }
}
```

Обратная операция — загрузка данных из файла построчно в `ListBox` — требует предварительной очистки текущего содержимого, чтобы избежать дублирования, и использования цикла с проверкой достижения конца файла (`EndOfStream`):

```csharp
listBox1.Items.Clear();

if (File.Exists("tasks.txt"))
{
    using (StreamReader reader = new StreamReader("tasks.txt", Encoding.UTF8))
    {
        string line;
        while ((line = reader.ReadLine()) != null)
        {
            listBox1.Items.Add(line);
        }
    }
}
```

### Роль кодировок и шаблона Disposable (конструкция `using`)

При работе с текстовыми файлами критически важно указывать кодировку (класс `Encoding`). Если файл сохранен в кодировке `UTF-8` (стандарт для современных приложений .NET 8), а программа пытается прочитать его в системной кодировке по умолчанию (`Encoding.Default`, например, `Windows-1251` для русскоязычной Windows), кириллические символы превращаются в «кракозябры» (нечитаемые символы).

> **Правило кодировок.** Всегда явно указывайте `Encoding.UTF8` при чтении и записи текстовых файлов, содержащих текст на разных языках, чтобы обеспечить переносимость данных между компьютерами.

Любой файловый поток (`Stream`, `StreamReader`, `StreamWriter`) является неуправляемым ресурсом операционной системы. Если открыть файл и не закрыть его, файл останется заблокированным до завершения работы программы. В C# для гарантированного освобождения ресурсов используется шаблон `IDisposable` и синтаксический сахар — конструкция `using`:

```csharp
using (StreamReader reader = new StreamReader(path, Encoding.UTF8))
{
    // Код чтения файла
} // Здесь автоматически вызывается метод reader.Dispose(), закрывающий поток и файл
```

Начиная с C# 8.0, доступна упрощенная форма записи `using` без фигурных скобок (так называемый *using declaration*), когда объект уничтожается при выходе из текущего метода:

```csharp
using StreamReader reader = new StreamReader(path, Encoding.UTF8);
string content = reader.ReadToEnd();
// Поток закроется в момент выхода из метода
```

### Контроль изменений в документе: флаг модификации (`isDirty`)

В профессиональных приложениях пользователь не должен терять данные при случайном закрытии окна или открытии нового файла поверх несохраненного. Для этого вводят логическую переменную состояния — флаг модификации (`isDirty`), который принимает значение `true`, если текст в редакторе был изменен, и `false`, если изменений не было.

Алгоритм работы с флагом:
1. При запуске программы или после успешного сохранения файла `isDirty = false`.
2. При любом изменении текста в `TextBox` (событие `TextChanged`) устанавливается `isDirty = true`.
3. При попытке закрыть форму или открыть другой файл проверяется значение `isDirty`. Если оно равно `true`, выводится диалоговое окно `MessageBox` с вопросом: «Сохранить изменения?».

---

### Пример 1. Разработка простого текстового редактора

**Условие:** Разработать оконное приложение (Windows Forms), представляющее собой текстовый редактор. Интерфейс должен содержать многострочный `TextBox` на всю форму, меню `MenuStrip` с пунктами «Файл -> Открыть», «Файл -> Сохранить» и «Файл -> Выход». Реализовать контроль изменений (`isDirty`), защиту от потери данных при закрытии и диалоговые окна `OpenFileDialog` и `SaveFileDialog`.

**Рисунок интерфейса (схемы):**
```text
+-------------------------------------------------------+
| Файл                                                  |
+-------------------------------------------------------+
| Открыть...      Ctrl+O                                |
| Сохранить       Ctrl+S                                |
| Выход           Alt+F4                                |
+-------------------------------------------------------+
|                                                       |
| [  Многострочный TextBox с полосами прокрутки       ] |
| [  (Multiline = true, ScrollBars = Vertical)        ] |
| [                                                     ] |
| [                                                     ] |
+-------------------------------------------------------+
```

**Ход рассуждений:**
1. Создадим форму `MainForm`, разместим на ней `MenuStrip` и `TextBox` со свойствами `Dock = DockStyle.Fill`, `Multiline = true`, `ScrollBars = ScrollBars.Vertical`.
2. Заведем поле класса `private bool isFileChanged = false;` и `private string currentFilePath = string.Empty;`.
3. В событии `TextChanged` у `TextBox` будем устанавливать `isFileChanged = true`.
4. Для открытия файла используем `OpenFileDialog`. Если пользователь выбрал файл и нажал «ОК», читаем его через `StreamReader` в `Encoding.UTF8`, записываем путь в `currentFilePath` и сбрасываем флаг `isFileChanged = false`.
5. Для сохранения проверяем, задан ли `currentFilePath`. Если нет — вызываем `SaveFileDialog`. Затем записываем текст через `StreamWriter`.
6. В событии закрытия формы `FormClosing` проверяем `isFileChanged`. Если текст изменен, спрашиваем пользователя о сохранении.

**Решение (код на C#):**

```csharp
using System;
using System.IO;
using System.Text;
using System.Windows.Forms;

namespace TextEditorApp
{
    public partial class MainForm : Form
    {
        private TextBox textBoxEditor;
        private MenuStrip menuStrip1;
        private ToolStripMenuItem fileMenu;
        private ToolStripMenuItem openMenuItem;
        private ToolStripMenuItem saveMenuItem;
        private ToolStripMenuItem exitMenuItem;

        private bool isFileChanged = false;
        private string currentFilePath = string.Empty;

        public MainForm()
        {
            InitializeComponentCustom();
        }

        private void InitializeComponentCustom()
        {
            this.Text = "Простой текстовый редактор";
            this.Width = 800;
            this.Height = 600;

            // Инициализация TextBox
            textBoxEditor = new TextBox();
            textBoxEditor.Multiline = true;
            textBoxEditor.ScrollBars = ScrollBars.Vertical;
            textBoxEditor.Dock = DockStyle.Fill;
            textBoxEditor.TextChanged += TextBoxEditor_TextChanged;
            this.Controls.Add(textBoxEditor);

            // Инициализация меню
            menuStrip1 = new MenuStrip();
            fileMenu = new ToolStripMenuItem("Файл");
            openMenuItem = new ToolStripMenuItem("Открыть...", null, OpenMenuItem_Click);
            saveMenuItem = new ToolStripMenuItem("Сохранить", null, SaveMenuItem_Click);
            exitMenuItem = new ToolStripMenuItem("Выход", null, ExitMenuItem_Click);

            fileMenu.DropDownItems.Add(openMenuItem);
            fileMenu.DropDownItems.Add(saveMenuItem);
            fileMenu.DropDownItems.Add(new ToolStripSeparator());
            fileMenu.DropDownItems.Add(exitMenuItem);

            menuStrip1.Items.Add(fileMenu);
            this.MainMenuStrip = menuStrip1;
            this.Controls.Add(menuStrip1);

            // Событие закрытия формы
            this.FormClosing += MainForm_FormClosing;
        }

        private void TextBoxEditor_TextChanged(object sender, EventArgs e)
        {
            if (!isFileChanged)
            {
                isFileChanged = true;
                this.Text = "Простой текстовый редактор *";
            }
        }

        private void OpenMenuItem_Click(object sender, EventArgs e)
        {
            if (!CheckSaveChanges()) return;

            using (OpenFileDialog openFileDialog = new OpenFileDialog())
            {
                openFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
                if (openFileDialog.ShowDialog() == DialogResult.OK)
                {
                    try
                    {
                        currentFilePath = openFileDialog.FileName;
                        using (StreamReader reader = new StreamReader(currentFilePath, Encoding.UTF8))
                        {
                            textBoxEditor.Text = reader.ReadToEnd();
                        }
                        isFileChanged = false;
                        this.Text = $"Простой текстовый редактор - {Path.GetFileName(currentFilePath)}";
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show($"Ошибка при чтении файла: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
        }

        private void SaveMenuItem_Click(object sender, EventArgs e)
        {
            SaveFile();
        }

        private bool SaveFile()
        {
            if (string.IsNullOrEmpty(currentFilePath))
            {
                using (SaveFileDialog saveFileDialog = new SaveFileDialog())
                {
                    saveFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
                    if (saveFileDialog.ShowDialog() == DialogResult.OK)
                    {
                        currentFilePath = saveFileDialog.FileName;
                    }
                    else
                    {
                        return false; // Пользователь отменил сохранение
                    }
                }
            }

            try
            {
                using (StreamWriter writer = new StreamWriter(currentFilePath, false, Encoding.UTF8))
                {
                    writer.Write(textBoxEditor.Text);
                }
                isFileChanged = false;
                this.Text = $"Простой текстовый редактор - {Path.GetFileName(currentFilePath)}";
                return true;
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Ошибка при записи файла: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
                return false;
            }
        }

        private bool CheckSaveChanges()
        {
            if (!isFileChanged) return true;

            DialogResult result = MessageBox.Show(
                "Файл был изменен. Сохранить изменения?",
                "Подтверждение",
                MessageBoxButtons.YesNoCancel,
                MessageBoxIcon.Warning);

            if (result == DialogResult.Yes)
            {
                return SaveFile();
            }
            else if (result == DialogResult.No)
            {
                return true;
            }
            else
            {
                return false; // Cancel
            }
        }

        private void ExitMenuItem_Click(object sender, EventArgs e)
        {
            this.Close();
        }

        private void MainForm_FormClosing(object sender, FormClosingEventArgs e)
        {
            if (!CheckSaveChanges())
            {
                e.Cancel = true; // Отменяем закрытие формы
            }
        }
    }
}
```

**Проверка:**
- При вводе текста в заглавии формы появляется символ звездочки `*`, что сигнализирует о наличии несохраненных данных.
- При попытке закрыть приложение выдается запрос `MessageBox`. Если нажать «Отмена», закрытие блокируется.
- Кодировка `UTF8` гарантирует корректную работу с русскими буквами. Сложность операций чтения и записи составляет $O(N)$, где $N$ — объем текста в символах.

`Ответ:` Разработан полноценный текстовый редактор с защитой от потери данных, поддержкой диалоговых окон и управлением кодировкой потоков.

---

### Пример 2. Экспорт и импорт списка задач (ListBox)

**Условие:** Разработать модуль управления списком задач (`ToDo List`). Приложение содержит `ListBox` для отображения задач, однострочный `TextBox` для ввода новой задачи, кнопки «Добавить», «Удалить», а также автоматическую загрузку списка из файла `tasks.txt` при старте программы и сохранение списка в этот же файл при закрытии формы.

**Рисунок интерфейса (схемы):}**
```text
+-------------------------------------------------------+
| Введите задачу: [ Купить хлеб       ] [ Добавить ]    |
|                                                       |
| +---------------------------------------------------+ |
| | 1. Купить хлеб                                    | |
| | 2. Сделать домашнее задание по C#                 | |
| | 3. Подготовиться к экзамену                       | |
| +---------------------------------------------------+ |
|                                       [ Удалить ]     |
+-------------------------------------------------------+
```

**Ход рассуждений:**
1. Для хранения задач используется элемент `ListBox`.
2. При загрузке формы (событие `Load`) проверяется существование файла `tasks.txt`. Если файл существует, построчно считываем его содержимое через `StreamReader` и добавляем в `listBoxTasks.Items`.
3. При закрытии формы (событие `FormClosing`) обходим коллекцию `listBoxTasks.Items` и записываем каждый элемент в файл `tasks.txt` с помощью `StreamWriter`.
4. Кнопки добавления и удаления модифицируют коллекцию `Items` в памяти, а сохранение происходит автоматически в фоновом режиме при закрытии.

**Решение (код на C#):**

```csharp
using System;
using System.IO;
using System.Text;
using System.Windows.Forms;

namespace TodoListApp
{
    public partial class TodoForm : Form
    {
        private ListBox listBoxTasks;
        private TextBox textBoxTaskInput;
        private Button buttonAdd;
        private Button buttonDelete;
        private const string FileName = "tasks.txt";

        public TodoForm()
        {
            InitializeComponentCustom();
        }

        private void InitializeComponentCustom()
        {
            this.Text = "Менеджер задач";
            this.Width = 500;
            this.Height = 400;

            // Поле ввода
            textBoxTaskInput = new TextBox();
            textBoxTaskInput.Location = new System.Drawing.Point(12, 15);
            textBoxTaskInput.Width = 330;
            this.Controls.Add(textBoxTaskInput);

            // Кнопка добавления
            buttonAdd = new Button();
            buttonAdd.Text = "Добавить";
            buttonAdd.Location = new System.Drawing.Point(355, 13);
            buttonAdd.Width = 115;
            buttonAdd.Click += ButtonAdd_Click;
            this.Controls.Add(buttonAdd);

            // ListBox для задач
            listBoxTasks = new ListBox();
            listBoxTasks.Location = new System.Drawing.Point(12, 50);
            listBoxTasks.Width = 458;
            listBoxTasks.Height = 260;
            this.Controls.Add(listBoxTasks);

            // Кнопка удаления
            buttonDelete = new Button();
            buttonDelete.Text = "Удалить выбранное";
            buttonDelete.Location = new System.Drawing.Point(315, 320);
            buttonDelete.Width = 155;
            buttonDelete.Click += ButtonDelete_Click;
            this.Controls.Add(buttonDelete);

            // Подписка на события жизненного цикла формы
            this.Load += TodoForm_Load;
            this.FormClosing += TodoForm_FormClosing;
        }

        private void ButtonAdd_Click(object sender, EventArgs e)
        {
            string taskText = textBoxTaskInput.Text.Trim();
            if (!string.IsNullOrEmpty(taskText))
            {
                listBoxTasks.Items.Add(taskText);
                textBoxTaskInput.Clear();
                textBoxTaskInput.Focus();
            }
            else
            {
                MessageBox.Show("Введите текст задачи!", "Внимание", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }

        private void ButtonDelete_Click(object sender, EventArgs e)
        {
            if (listBoxTasks.SelectedItem != null)
            {
                listBoxTasks.Items.Remove(listBoxTasks.SelectedItem);
            }
            else
            {
                MessageBox.Show("Выберите задачу для удаления!", "Внимание", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }

        private void TodoForm_Load(object sender, EventArgs e)
        {
            // Автоматическая загрузка списка при старте
            if (File.Exists(FileName))
            {
                try
                {
                    using (StreamReader reader = new StreamReader(FileName, Encoding.UTF8))
                    {
                        string line;
                        while ((line = reader.ReadLine()) != null)
                        {
                            if (!string.IsNullOrWhiteSpace(line))
                            {
                                listBoxTasks.Items.Add(line);
                            }
                        }
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"Не удалось загрузить задачи: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
                }
            }
        }

        private void TodoForm_FormClosing(object sender, FormClosingEventArgs e)
        {
            // Автоматическое сохранение списка при закрытии
            try
            {
                using (StreamWriter writer = new StreamWriter(FileName, false, Encoding.UTF8))
                {
                    foreach (var item in listBoxTasks.Items)
                    {
                        writer.WriteLine(item.ToString());
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Не удалось сохранить задачи: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }
}
```

**Проверка:**
- При запуске программы в списке автоматически появляются ранее добавленные задачи из файла `tasks.txt`. Если файла нет, программа запускается с пустым списком без генерации исключений благодаря проверке `File.Exists(FileName)`.
- При добавлении и удалении элементы корректно обновляются в интерфейсе.
- При закрытии окна все элементы сохраняются построчно в кодировке `UTF-8`. Сложность загрузки и сохранения линейно зависит от количества элементов в `ListBox`: $O(K)$, где $K$ — количество строк.

`Ответ:` Реализовано оконное приложение для управления списком задач с автоматической персистентностью данных при запуске и закрытии формы с использованием потоков `StreamReader` и `StreamWriter`.

## 3. Бинарные файлы и табличные данные в DataGridView

### Бинарные файлы и табличные данные в DataGridView

Текстовые форматы файлов, такие как `.txt` или `.csv`, удобны для чтения человеком и для обмена данными с другими программами, однако они неэффективны для хранения сложных или строго типизированных структур. Когда приложение работает с табличными данными, содержащими смесь строк, чисел с плавающей точкой и логических флагов, текстовое представление требует постоянного преобразования типов (парсинга), что замедляет работу и увеличивает размер файлов на диске.

Для эффективного хранения и быстрой загрузки структурированных данных в приложениях Windows Forms применяются бинарные файлы. В паре с визуальным компонентом `DataGridView` и коллекцией `BindingList<T>` это позволяет организовать полноценное локальное хранилище данных без использования тяжеловесных СУБД.

---

### Особенности бинарного ввода-вывода и классы BinaryWriter / BinaryReader

> **Определение.** Бинарный файл — это файл, данные в котором хранятся в том же двоичном формате, в котором они располагаются в оперативной памяти компьютера, без промежуточного преобразования в символы текста.

Для записи и чтения примитивных типов данных в байтовом потоке в платформе .NET предусмотрены специальные классы-обертки: `BinaryWriter` и `BinaryReader`. Они работают поверх базового класса потока `Stream` (обычно `FileStream`), который осуществляет непосредственный доступ к файлу на жестком диске.

Таблица ниже сопоставляет методы записи и чтения основных типов данных, используемых в табличных моделях.

| Тип данных в C# | Метод записи (`BinaryWriter`) | Метод чтения (`BinaryReader`) | Размер в байтах |
| :--- | :--- | :--- | :--- |
| `int` | `Write(int value)` | `ReadInt32()` | 4 |
| `double` | `Write(double value)` | `ReadDouble()` | 8 |
| `bool` | `Write(bool value)` | `ReadBoolean()` | 1 |
| `string` | `Write(string value)` | `ReadString()` | Переменный (включает длину) |

Главное правило работы с бинарными потоками: **порядок чтения данных должен строго соответствовать порядку их записи**. Если в файл сначала записали строку, а затем целое число, то и считывать их из файла необходимо в той же последовательности. Нарушение порядка приведет к искажению данных (десихронизации потока) или генерации исключения `EndOfStreamException`.

---

### Связь DataGridView с коллекциями через BindingList\<T\>

Компонент `DataGridView` предназначен для отображения табличных данных. Самый надежный и профессиональный способ наполнения этой таблицы данными в ООП-программировании — это использование паттерна «источник данных» (Data Binding).

Вместо того чтобы вручную добавлять строки в визуальный компонент через метод `Rows.Add()`, мы создаем типизированную коллекцию объектов `BindingList<T>` и привязываем ее к таблице.

> **Определение.** `BindingList<T>` — это специализированная обобщенная коллекция в .NET, которая реализует интерфейсы уведомления об изменениях (`IBindingList`, `INotifyPropertyChanged`). Когда элемент добавляется в коллекцию или удаляется из нее, `DataGridView` автоматически обновляет свой визуальный интерфейс без ручного перезапуска отрисовки.

Для демонстрации создадим структуру `Product`, описывающую товар на складе:

```csharp
public struct Product
{
    public string Name { get; set; }
    public double Price { get; set; }
    public int Quantity { get; set; }

    public Product(string name, double price, int quantity)
    {
        Name = name;
        Price = price;
        Quantity = quantity;
    }
}
```

Чтобы связать эту структуру с компонентом `DataGridView` на форме, выполняются следующие шаги:
1. Создается экземпляр списка: `BindingList<Product> products = new BindingList<Product>();`
2. Список назначается источником данных таблицы: `dataGridView1.DataSource = products;`
3. Все изменения в списке `products` (добавление новых товаров, удаление) мгновенно отображаются в таблице.

---

### Построчное чтение, запись и контроль целостности потока

При сохранении таблицы в бинарный файл процесс строится следующим образом:
1. Сначала записывается общее количество записей в коллекции (тип `int`), чтобы при чтении знать, сколько итераций цикла нужно выполнить.
2. Затем в цикле для каждого объекта из коллекции последовательно записываются его поля (`Name`, `Price`, `Quantity`).

При чтении алгоритм зеркален:
1. Считывается количество записей (`int`).
2. В цикле считываются поля каждого товара, формируется объект `Product`, который добавляется в `BindingList<Product>`.

Для предотвращения зависаний и аварийных завершений программы при чтении поврежденных или усеченных файлов используется проверка конца потока. У любого потока (`Stream`), доступного через свойство `BaseStream` у `BinaryReader`, есть два ключевых свойства:
- `Length` — общая длина файла в байтах.
- `Position` — текущая позиция указателя чтения в байтах.

Условие безопасности при чтении: `reader.BaseStream.Position < reader.BaseStream.Length`. Если позиция достигла длины файла, дальнейшее чтение вызовет ошибку.

---

### Пример 1. Сохранение таблицы товаров в бинарный файл

Рассмотрим полную реализацию задачи. Пользователь нажимает кнопку «Сохранить», открывается стандартное окно сохранения файла `SaveFileDialog`, после чего все данные из `BindingList<Product>`, привязанного к `DataGridView`, сериализуются в бинарный файл.

```csharp
using System;
using System.ComponentModel;
using System.IO;
using System.Windows.Forms;

namespace WarehouseApp
{
    public partial class MainForm : Form
    {
        // Коллекция данных, привязанная к таблице
        private BindingList<Product> productList;

        public MainForm()
        {
            InitializeComponent();
            
            // Инициализация тестовыми данными
            productList = new BindingList<Product>()
            {
                new Product("Ноутбук", 75000.50, 10),
                new Product("Мышь проводная", 1200.00, 55),
                new Product("Монитор", 18500.75, 8)
            };

            // Связывание источника данных с компонентом DataGridView на форме
            dataGridView1.DataSource = productList;
        }

        private void btnSaveBinary_Click(object sender, EventArgs e)
        {
            // Создаем и настраиваем диалоговое окно сохранения файла
            using (SaveFileDialog saveDlg = new SaveFileDialog())
            {
                saveDlg.Filter = "Бинарные файлы (*.bin)|*.bin|Все файлы (*.*)|*.*";
                saveDlg.Title = "Сохранение базы данных склада";
                saveDlg.DefaultExt = "bin";

                // Если пользователь выбрал файл и нажал ОК
                if (saveDlg.ShowDialog() == DialogResult.OK)
                {
                    try
                    {
                        // Открываем потоковый писатель с использованием конструкции using для гарантированного закрытия файла
                        using (BinaryWriter writer = new BinaryWriter(File.Open(saveDlg.FileName, FileMode.Create)))
                        {
                            // 1. Записываем общее количество элементов в коллекции
                            writer.Write(productList.Count);

                            // 2. Поочередно записываем каждый объект товара
                            foreach (var prod in productList)
                            {
                                writer.Write(prod.Name);
                                writer.Write(prod.Price);
                                writer.Write(prod.Quantity);
                            }
                        }

                        MessageBox.Show("Данные успешно сохранены в бинарный файл!", 
                                        "Успех", MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    catch (IOException ioEx)
                    {
                        MessageBox.Show($"Ошибка ввода-вывода при записи файла: {ioEx.Message}", 
                                        "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                    catch (UnauthorizedAccessException unAuthEx)
                    {
                        MessageBox.Show($"Отказано в доступе к директории: {unAuthEx.Message}", 
                                        "Ошибка безопасности", MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
        }
    }
}
```

#### Пояснение по блокам кода:
1. **`BindingList<Product>`**: Обеспечивает синхронизацию памяти приложения и графического элемента `DataGridView`.
2. **`FileMode.Create`**: Указывает операционной системе создать новый файл или полностью перезаписать существующий с таким же именем.
3. **`BinaryWriter.Write(...)`**: Последовательно упаковывает строку (в кодировке UTF-8 с префиксом длины), число с плавающей точкой (`double`, 8 байт) и целое число (`int`, 4 байта) в двоичный поток.
4. **Конструкция `using (...)`**: Гарантирует вызов метода `Close()` и освобождение дескрипторов файла операционной системой даже в случае возникновения исключений.

---

### Пример 2. Загрузка базы данных из бинарного файла с проверкой целостности

Теперь рассмотрим обратную операцию — чтение бинарного файла, восстановление коллекции `BindingList<Product>` и передача ее в `DataGridView`. Особое внимание здесь уделено проверке формата файла и контролю конца потока.

```csharp
private void btnLoadBinary_Click(object sender, EventArgs e)
{
    using (OpenFileDialog openDlg = new OpenFileDialog())
    {
        openDlg.Filter = "Бинарные файлы (*.bin)|*.bin|Все файлы (*.*)|*.*";
        openDlg.Title = "Загрузка базы данных склада";

        if (openDlg.ShowDialog() == DialogResult.OK)
        {
            try
            cheked // Проверка формата через блок try-catch и логику чтения
            {
                BindingList<Product> tempLoadedList = new BindingList<Product>();

                using (BinaryReader reader = new BinaryReader(File.Open(openDlg.FileName, FileMode.Open)))
                {
                    // Проверяем, есть ли вообще данные в файле (минимум 4 байта для счетчика)
                    if (reader.BaseStream.Length < 4)
                    {
                        throw new FormatException("Выбранный файл поврежден или имеет неверный формат (пустой файл).");
                    }

                    // 1. Читаем количество элементов
                    int count = reader.ReadInt32();

                    // Защита от отрицательных или неправдоподобных значений счетчика в поврежденных файлах
                    if (count < 0 || count > 1000000)
                    {
                        throw new FormatException("Некорректная структура данных в файле.");
                    }

                    // 2. Цикл чтения записей с проверкой целостности потока
                    for (int i = 0; i < count; i++)
                    {
                        // Проверяем, не достигли ли мы конца файла раньше времени
                        if (reader.BaseStream.Position >= reader.BaseStream.Length)
                        {
                            throw new EndOfStreamException("Файл обрван: реальное количество записей меньше заявленного в заголовке.");
                        }

                        string name = reader.ReadString();
                        double price = reader.ReadDouble();
                        int quantity = reader.ReadInt32();

                        tempLoadedList.Add(new Product(name, price, quantity));
                    }
                }

                // Если все данные успешно прочитаны, заменяем текущую коллекцию
                productList = tempLoadedList;
                dataGridView1.DataSource = productList;

                MessageBox.Show($"Успешно загружено записей: {productList.Count}", 
                                "Загрузка завершена", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
            catch (FormatException formatEx)
            {
                MessageBox.Show($"Ошибка формата файла: {formatEx.Message}", 
                                "Ошибка данных", MessageBoxButtons.OK, MessageBoxIcon.Warning);
            }
            catch (EndOfStreamException eosEx)
            {
                MessageBox.Show($"Ошибка целостности файла: {eosEx.Message}", 
                                "Ошибка чтения", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
            catch (IOException ioEx)
            {
                MessageBox.Show($"Ошибка при чтении файла: {ioEx.Message}", 
                                "Ошибка I/O", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }
}
```

#### Разбор логики и проверки целостности:
1. **Проверка `reader.BaseStream.Length < 4`**: Предотвращает попытку чтения из пустого файла, где отсутствует даже заголовок с количеством элементов.
2. **Валидация счетчика `count`**: Ограничение `0 < count < 1000000` защищает программу от сценария, когда случайный бинарный файл (например, музыкальный трек или картинка) открывают как базу данных, и случайные байты интерпретируются как гигантское число элементов, что вызвало бы переполнение памяти (`OutOfMemoryException`).
3. **Проверка `reader.BaseStream.Position >= reader.BaseStream.Length` внутри цикла**: Гарантирует, что если файл был искусственно обрезан злоумышленником или поврежден при передаче, программа не упадет с системной ошибкой `EndOfStreamException`, а перехватит ее в блоке `catch` и выведет понятное пользователю сообщение.

---

### Контрольные вопросы для самопроверки

1. Почему для работы со структурированными числовыми и текстовыми данными в бинарном файле предпочтительнее использовать `BinaryWriter`/`BinaryReader`, а не стандартный `StreamWriter`/`StreamReader`?
2. К чему приведет ситуация, если при чтении бинарного файла сначала вызвать метод `ReadInt32()`, а затем `ReadDouble()`, в то время как при записи эти типы данных следовали в обратном порядке?
3. Какую роль выполняет класс `BindingList<T>` при связывании данных из файла с компонентом `DataGridView`?
4. Как с помощью свойств `BaseStream.Position` и `BaseStream.Length` защитить приложение от зависания при чтении поврежденного файла?

## 4. Обработка исключений и обеспечение отказоустойчивости при файловых операциях

### Классификация исключений ввода-вывода в . пространство имен System.IO

При разработке оконных приложений на платформе .NET 8 с использованием C# работа с файловой системой сопряжена с множеством внешних факторов, которые программа не может контролировать напрямую. Пользователь может попытаться открыть файл, который был удален или перемещен, на диске может закончиться свободное место, либо у учетной записи пользователя не будет прав на чтение или запись в выбранную директорию. Если такие ситуации не обрабатывать явно, приложение аварийно завершит свою работу с необработанным исключением (Unhandled Exception), что приведет к потере несохраненных данных и негативно скажется на пользовательском опыте.

Все операции ввода-вывода в .NET базируются на пространстве имен `System.IO`. При возникновении ошибок среды выполнения генерируют исключения, наследуемые от базового класса `System.IO.IOException` или общего класса `System.Exception`. 

> **Определение.** Исключение (Exception) — это объект, представляющий ошибку или непредвиденную ситуацию, которая возникает во время выполнения программы. Механизм обработки исключений позволяет перехватить этот объект и выполнить альтернативный сценарий вместо аварийного завершения процесса.

Рассмотрим основные классы исключений, с которыми вы неизбежно столкнетесь при работе с файлами в Windows Forms:

1. `System.IO.IOException` — базовый класс для всех исключений ввода-вывода. Возникает при общих проблемах с устройствами хранения (например, ошибка чтения с поврежденного флеш-накопителя или когда файл заблокирован другим процессом).
2. `System.IO.FileNotFoundException` — специализированное исключение, которое генерируется, когда предпринимается попытка открыть несуществующий по указанному пути файл для чтения.
3. `System.IO.DirectoryNotFoundException` — возникает, если в указанном пути к файлу отсутствует одна или несколько промежуточных директорий (например, папка была удалена сторонней программой).
4. `System.UnauthorizedAccessException` — выбрасывается операционной системой, когда у текущего пользователя недостаточно прав доступа для чтения, записи или создания файла (например, попытка записать файл в системную папку `C:\Windows\System32` без прав администратора).
5. `System.ArgumentException` и `System.ArgumentNullException` — возникают, если в качестве пути к файлу передана пустая строка (`""`), `null` или строка, содержащая недопустимые символы (например, `<`, `>`, `?`, `*`).

В иерархии классов исключений более специфичные классы стоят ближе к источнику ошибки, чем общие. Это важнейшее правило при построении блоков перехвата ошибок.

---

### Структура безопасного блока try-catch-finally при доступе к файловой системе

Для предотвращения падения приложения используется конструкция `try-catch-finally`. Ее синтаксис в C# выглядит следующим образом:

```csharp
try
{
    // Код, который может потенциально вызвать исключение ввода-вывода
    // (например, открытие потока, чтение или запись файла)
}
고자(FileNotFoundException ex)
{
    // Обработка конкретной ситуации: файл не найден
}
catch (UnauthorizedAccessException ex)
{
    // Обработка ошибки доступа
}
catch (IOException ex)
{
    // Обработка любых других ошибок ввода-вывода
}
catch (Exception ex)
{
    // Перехват вообще всех остальных непредвиденных исключений
}
finally
{
    // Блок, который выполнится ВСЕГДА: и при успешном выполнении, и при ошибке.
    // Здесь освобождаются ресурсы (закрытие потоков, удаление временных файлов)
}
```

Принцип работы блока `try-catch-finally`:
1. Среда выполнения начинает исполнять код внутри блока `try`.
2. Если строка кода выбрасывает исключение (например, `new FileNotFoundException()`), выполнение немедленно прерывается, а оставшаяся часть блока `try` пропускается.
3. Управление передается по цепочке блоков `catch` сверху вниз. Среда ищет первый блок `catch`, тип которого совпадает с типом выброшенного исключения (или является его базовым классом).
4. **Важное правило порядка catch-блоков:** Сначала всегда перечисляются самые специфичные (узкие) исключения (`FileNotFoundException`, `DirectoryNotFoundException`), затем более общие (`IOException`), и в самом конце — универсальный `Exception`. Если поставить `Exception` на первое место, компилятор выдаст ошибку, так как последующие узкие блоки никогда не получат управления.
5. Блок `finally` выполняется гарантированно после завершения блоков `try` или `catch`. В современных версиях C# с поддержкой конструкции `using` необходимость писать ручное закрытие потоков в `finally` снизилась, однако этот блок незаменим для сброса состояния интерфейса (например, выключения индикатора загрузки).

---

### Информирование пользователя через MessageBox

В консольных приложениях ошибку можно вывести в стандартный поток вывода (`Console.WriteLine`). В графических приложениях Windows Forms единственным правильным способом сообщить пользователю о проблеме является вызов модального диалогового окна `MessageBox.Show()`.

> **Определение.** `MessageBox` — это стандартный класс Windows Forms, предназначенный для отображения всплывающих модальных окон с текстом сообщения, заголовком, значками и кнопками управления. Модальность окна означает, что пользователь не сможет продолжить работу с главной формой приложения, пока не закроет это диалоговое окно.

Синтаксис вызова метода для информирования об ошибке:
```csharp
MessageBox.Show(
    "Текст ошибки с понятным описанием", 
    "Заголовок окна", 
    MessageBoxButtons.OK, 
    MessageBoxIcon.Error
);
```

Разбор параметров метода `MessageBox.Show`:
- Первый параметр (`string text`) — подробное описание проблемы на русском языке, понятное обычному пользователю (технический стекtrace показывать пользователю нельзя).
- Второй параметр (`string caption`) — заголовок окна (например, «Ошибка файла», «Внимание»).
- Третий параметр (`MessageBoxButtons buttons`) — набор кнопок (обычно `MessageBoxButtons.OK` для уведомлений).
- Четвертый параметр (`MessageBoxIcon icon`) — значок, визуально определяющий тип сообщения (`MessageBoxIcon.Error` — для критических ошибок, `MessageBoxIcon.Warning` — для предупреждений, `MessageBoxIcon.Information` — для успешных операций).

---

### Проверка существования файла и прав доступа перед выполнением операций

Помимо перехвата исключений после их возникновения, хорошей практикой программирования является предварительная проверка условий с помощью вспомогательных статических методов класса `System.IO.File`. Это позволяет избежать лишних затрат ресурсов на генерацию исключений там, где ситуацию можно предвидеть.

Основные методы класса `File`:
- `File.Exists(string path)` — возвращает логическое значение `true`, если файл по указанному пути существует и доступен, и `false` в противном случае.
- `File.Directory.Exists(string path)` — проверяет существование директории.

Однако важно помнить про так называемые состояния гонки (Race Conditions): даже если метод `File.Exists` вернул `true`, между моментом проверки и моментом открытия файла другой процесс может этот файл удалить или заблокировать. Поэтому **предварительная проверка не отменяет необходимость использования блока `try-catch`**, а лишь дополняет его.

---

### Предотвращение зависания пользовательского интерфейса при работе с большими файлами

В приложениях Windows Forms весь графический интерфейс (отрисовка элементов, реакция на клики мыши и нажатия клавиш) работает в рамках одного единственного потока, который называется **UI-потоком** (главным потоком приложения). 

Если ваша программа пытается прочитать из файла или записать на диск огромный объем данных (например, файл размером 500 мегабайт) с помощью синхронных методов вроде `File.ReadAllText` или циклов чтения в главном потоке, этот поток оказывается полностью заблокирован операцией ввода-вывода. Для операционной системы это выглядит так, будто приложение зависло: окно перестает перерисовываться, кнопка закрытия «крестик» не реагирует, а в заголовке окна появляется надпись «Не отвечает».

Способы предотвращения зависания интерфейса:
1. Использование асинхронного программирования с ключевыми словами `async` и `await` (например, методы `StreamReader.ReadToEndAsync()`, `File.WriteAllTextAsync()`). Это позволяет освободить UI-поток на время ожидания ответа от жесткого диска.
2. Ограничение максимального размера обрабатываемых файлов на уровне приложения с предварительной проверкой свойства `FileInfo.Length`.
3. Отображение элементов визуализации процесса (например, компонента `ProgressBar`) при пакетной обработке данных.

---

### Разобранные примеры

#### Пример 1. Обработка попытки открытия несуществующего файла или файла, заблокированного другим приложением

**Условие задачи:** Разработать метод для формы Windows Forms, который при нажатии на кнопку «Открыть отчет» с помощью `OpenFileDialog` позволяет выбрать текстовый файл, а затем считывает его содержимое в многострочный `TextBox`. Необходимо предусмотреть всевозможные аварийные ситуации: файл не выбран, файл не существует, файл заблокирован другим приложением (например, открыт в Microsoft Word с эксклюзивной блокировкой) или у пользователя нет прав на чтение.

**Рисунок словами (интерфейс):**
На форме располагается:
- Кнопка `btnOpen` с текстом «Открыть отчет...» (в левом верхнем углу).
- Многострочный элемент `TextBox` с именем `txtContent`, растянутый на всю центральную часть формы (свойства `Multiline = true`, `ScrollBars = ScrollBars.Vertical`).

**Ход рассуждений:**
1. Сначала вызываем диалоговое окно `OpenFileDialog`. Если пользователь нажал кнопку «Отмена», прерываем выполнение метода.
2. Проверяем физическое существование файла с помощью `File.Exists()`, чтобы сразу выдать точечное сообщение, если файл был удален сразу после выбора.
3. Оборачиваем процедуру создания потока `StreamReader` и чтения данных в блок `try-catch`.
4. В блоке `catch` перехватываем специфические исключения: `FileNotFoundException`, `UnauthorizedAccessException`, общий `IOException` для блокировок и `Exception` для непредвиденных сбоев. Для каждого случая формируем понятный пользователю `MessageBox`.

**Решение в общем виде и код на C#:**

```csharp
using System;
using System.IO;
using System.Windows.Forms;

namespace FileAppDemo
{
    public partial class MainForm : Form
    {
        public MainForm()
        {
            InitializeComponent();
        }

        private void btnOpen_Click(object sender, EventArgs e)
        {
            // Шаг 1. Настройка и вызов диалогового окна выбора файла
            using (OpenFileDialog openFileDialog = new OpenFileDialog())
            {
                openFileDialog.Title = "Выберите файл отчета";
                openFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
                openFileDialog.RestoreDirectory = true;

                // Если пользователь нажал «Отмена», прерываем работу метода
                if (openFileDialog.ShowDialog() != DialogResult.OK)
                {
                    return;
                }

                string filePath = openFileDialog.FileName;

                // Шаг 2. Дополнительная проверка существования файла
                if (!File.Exists(filePath))
                {
                    MessageBox.Show(
                        "Выбранный файл больше не существует на диске.",
                        "Ошибка файла",
                        MessageBoxButtons.OK,
                        MessageBoxIcon.Warning
                    );
                    return;
                }

                // Шаг 3. Безопасное чтение файла с обработкой исключений
                try
                {
                    // Конструкция using гарантирует закрытие потока даже при ошибке чтения
                    using (StreamReader reader = new StreamReader(filePath))
                    {
                        // Читаем весь текст из файла в память
                        string fileText = reader.ReadToEnd();
                        
                        // Выводим текст в многострочный TextBox
                        txtContent.Text = fileText;
                    }

                    // Сообщение об успешном завершении (опционально)
                    // MessageBox.Show("Файл успешно загружен!", "Успех", MessageBoxButtons.OK, MessageBoxIcon.Information);
                }
                catch (FileNotFoundException)
                {
                    MessageBox.Show(
                        "Не удалось найти указанный файл.", 
                        "Ошибка чтения", 
                        MessageBoxButtons.OK, 
                        MessageBoxIcon.Error
                    );
                }
                catch (UnauthorizedAccessException)
                {
                    MessageBox.Show(
                        "У вас нет прав на чтение этого файла. Обратитесь к системному администратору.", 
                        "Доступ запрещен", 
                        MessageBoxButtons.OK, 
                        MessageBoxIcon.Error
                    );
                }
                catch (IOException ioEx)
                {
                    // Это исключение сработает, если файл заблокирован другой программой
                    MessageBox.Show(
                        $"Файл заблокирован другим приложением или поврежден.\n\nТехническая детализация: {ioEx.Message}", 
                        "Ошибка ввода-вывода", 
                        MessageBoxButtons.OK, 
                        MessageBoxIcon.Error
                    );
                }
                catch (Exception ex)
                {
                    // Перехват любых других непредвиденных сбоев
                    MessageBox.Show(
                        $"Произошла непредвиденная ошибка: {ex.Message}", 
                        "Критическая ошибка", 
                        MessageBoxButtons.OK, 
                        MessageBoxIcon.Error
                    );
                }
            }
        }
    }
}
```

**Проверка и правдоподобность:**
- Единицы измерения и типы данных: пути обрабатываются как строки (`string`), исключения типизированы.
- Правдоподобность: если открыть файл в Microsoft Word и попытаться загрузить его этой программой, сработает блок `IOException`, сообщив пользователю о блокировке процесса, программа не упадет.

---

#### Пример 2. Защита программы от сбоя при попытке записи в защищенную от записи директорию

**Условие задачи:** Разработать метод для сохранения текстовых данных из `TextBox` в файл конфигурации `settings.dat`. Если пользователь попытается сохранить файл в защищенную системную директорию (например, в корень диска `C:\` или в системную папку Windows), программа должна перехватить исключение `UnauthorizedAccessException`, предложить пользователю выбрать другую папку через диалоговое окно сохранения (`SaveFileDialog`) и повторить попытку записи.

**Рисунок словами (интерфейс):**
На форме располагается:
- Кнопка `btnSave` с текстом «Сохранить настройки».
- Поле ввода `txtSettingsInput` для ввода параметров.

**Ход рассуждений:**
1. Попытаемся записать файл в жестко заданную директорию, которая с высокой вероятностью защищена от записи для обычного пользователя (например, `C:\ProtectedConfig\settings.dat`).
2. Обернем операцию записи в блок `try-catch`.
3. В случае возникновения `UnauthorizedAccessException` выводим предупреждение с помощью `MessageBox` и вызываем `SaveFileDialog`, чтобы пользователь мог перенаправить сохранение в безопасное место (например, в папку документов пользователя `Environment.SpecialFolder.MyDocuments`).
4. Повторяем попытку записи по новому пути.

**Решение в общем виде и код на C#:**

```csharp
using System;
using System.IO;
using System.Windows.Forms;

namespace FileAppDemo
{
    public partial class MainForm : Form
    {
        public MainForm()
        {
            InitializeComponent();
        }

        private void btnSave_Click(object sender, EventArgs e)
        {
            string defaultPath = @"C:\ProtectedConfig\settings.dat";
            string contentToSave = txtSettingsInput.Text;

            try
            {
                // Попытка прямой записи в защищенную директорию
                WriteDataToFile(defaultPath, contentToSave);
                
                MessageBox.Show(
                    "Настройки успешно сохранены!", 
                    "Сохранение", 
                    MessageBoxButtons.OK, 
                    MessageBoxIcon.Information
                );
            }
            catch (UnauthorizedAccessException)
            {
                // Обработка ошибки доступа: предлагаем пользователю выбрать другое место
                DialogResult result = MessageBox.Show(
                    "Недостаточно прав для сохранения файла в системную паку.\nХотите выбрать другую папку для сохранения?", 
                    "Ошибка доступа", 
                    MessageBoxButtons.YesNo, 
                    MessageBoxIcon.Warning
                );

                if (result == DialogResult.Yes)
                {
                    using (SaveFileDialog saveDlg = new SaveFileDialog())
                    {
                        saveDlg.Title = "Сохранить файл настроек";
                        saveDlg.Filter = "Файлы данных (*.dat)|*.dat|Все файлы (*.*)|*.*";
                        saveDlg.FileName = "settings.dat";
                        
                        // Предлагаем папку Документы в качестве начальной
                        saveDlg.InitialDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);

                        if (saveDlg.ShowDialog() == DialogResult.OK)
                        {
                            try
                            {
                                // Повторная попытка записи по безопасному пути, выбранному пользователем
                                WriteDataToFile(saveDlg.FileName, contentToSave);

                                MessageBox.Show(
                                    "Файл успешно сохранен по новому пути!", 
                                    "Успех", 
                                    MessageBoxButtons.OK, 
                                    MessageBoxIcon.Information
                                );
                            }
                            catch (Exception ex)
                            {
                                MessageBox.Show(
                                    $"Не удалось сохранить файл: {ex.Message}", 
                                    "Критическая ошибка", 
                                    MessageBoxButtons.OK, 
                                    MessageBoxIcon.Error
                                );
                            }
                        }
                    }
                }
            }
            catch (DirectoryNotFoundException)
            {
                MessageBox.Show(
                    "Указанная директория не существует. Проверьте правильность пути.", 
                    "Ошибка пути", 
                    MessageBoxButtons.OK, 
                    MessageBoxIcon.Error
                );
            }
            catch (IOException ioEx)
            {
                MessageBox.Show(
                    $"Ошибка ввода-вывода при записи файла: {ioEx.Message}", 
                    "Ошибка диска", 
                    MessageBoxButtons.OK, 
                    MessageBoxIcon.Error
                );
            }
        }

        // Вспомогательный метод для инкапсуляции логики записи
        private void WriteDataToFile(string path, string data)
        {
            // Автоматическое создание директории, если она отсутствует (для безопасных путей)
            string directoryName = Path.GetDirectoryName(path);
            if (!string.IsNullOrEmpty(directoryName) && !Directory.Exists(directoryName))
            {
                Directory.CreateDirectory(directoryName);
            }

            // Запись данных в файл в кодировке UTF-8
            File.WriteAllText(path, data);
        }
    }
}
```

**Проверка и правдоподобность:**
- Исключение `UnauthorizedAccessException` перехвачено корректно. Пользователь не видит технических деталей трассировки стека, а получает четкую инструкцию на русском языке с возможностью исправить ситуацию «на лету» с помощью стандартного диалогового окна.
- Использован вспомогательный метод `WriteDataToFile` для устранения дублирования кода записи, что соответствует стандартам структурного программирования.

---

### Ответы для самопроверки
- Какой класс является базовым для всех исключений файловой системы в .NET? (`System.IO.IOException`)
- Почему блоки `catch` для специфических исключений (`FileNotFoundException`) должны располагаться выше общего блока `Exception`? (Потому что среда выполнения выбирает первый подходящий блок сверху вниз; если поставить `Exception` выше, он перехватит все ошибки, и компилятор выдаст предупреждение о недостижимости кода).
- Какой метод класса `File` позволяет проверить существование файла перед его открытием без вызова исключения? (`File.Exists(path)`).

## 5. Автоматическое сохранение настроек приложения и сессий

### Организация конфигурационных файлов и путей к данным в .NET 8

При разработке оконных приложений на платформе .NET 8 с использованием Windows Forms часто возникает необходимость сохранять не только документы пользователя, но и служебную информацию: размеры и положение окна на экране, выбранные настройки интерфейса, последние открытые файлы или состояние сессии. Такие данные принято хранить в конфигурационных файлах.

В современных приложениях стандартом де-факто для конфигурации стал текстовый формат JSON (JavaScript Object Notation). Для работы с ним в .NET 8 встроен высокопроизводительный и удобный механизм сериализации и десериализации объектов — пространство имен `System.Text.Json`.

> **Определение.** Сериализация — это процесс преобразования объекта в памяти (например, экземпляра класса или структуры) в последовательность байт или текстовый формат (например, JSON) для сохранения в файл или передачи по сети. Десериализация — это обратный процесс восстановления объекта из сохраненного представления.

Перед тем как сохранять конфигурацию, необходимо определить, **куда именно** её записывать. Запрещено сохранять файлы настроек в корневой каталог установленной программы (например, в `C:\Program Files\...`), так как современная операционная система Windows защищает эту папку и обычный пользователь (или запущенное им приложение без прав администратора) не сможет записать туда файл, что приведет к исключению `UnauthorizedAccessException`.

Для хранения пользовательских данных в .NET предусмотрен специальный перечислений класс `Environment.SpecialFolder`, который позволяет программно получить путь к системным папкам текущего пользователя независимо от версии Windows и имени учетной записи.

Основные системные папки, используемые при разработке приложений:
- `Environment.SpecialFolder.ApplicationData` — папка `Roaming` (обычно путь выглядит как `C:\Users\ИмяПользователя\AppData\Roaming`). Содержимое этой папки синхронизируется в доменных сетях Windows, поэтому она идеально подходит для небольших конфигурационных файлов и настроек.
- `Environment.SpecialFolder.LocalApplicationData` — папка `Local` (`C:\Users\ИмяПользователя\AppData\Local`). Предназначена для больших файлов кэша и данных, которые не нужно синхронизировать.
- `Environment.SpecialFolder.MyDocuments` — папка «Мои документы», используемая для сохранения пользовательских документов (текстовых файлов, таблиц, отчетов).

Для формирования полного пути к файлу настроек используется статический метод `Path.Combine(string path1, string path2)`, который автоматически подставляет правильные разделители путей (`\` для Windows), предотвращая ошибки.

Пример получения пути к файлу конфигурации `settings.json`:

```csharp
string folderPath = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
// Создаем подкаталог для нашего приложения, если он еще не создан
string appFolder = Path.Combine(folderPath, "MyNotesApp");
Directory.CreateDirectory(appFolder);
string settingsFilePath = Path.Combine(appFolder, "settings.json");
```

---

### Жизненный цикл формы: события Load и FormClosing

Чтобы настройки приложения применялись при запуске и сохранялись при закрытии, используются ключевые события класса `Form`:

1. **Событие `Load`** (обрабатывается методом `Form_Load`) — генерируется в момент, когда форма загружается в память и отображается на экране, но пользователь её еще не видит. Это идеальное место для чтения файла конфигурации, восстановления размеров окна, позиций элементов и загрузки пользовательских данных.
2. **Событие `FormClosing`** (обрабатывается методом `Form_Closing`) — генерируется в момент, когда пользователь пытается закрыть окно (нажимает крестик `[X]`, комбинацию клавиш `Alt+F4` или вызывает закрытие через код). В обработчике этого события программа успевает перехватить момент закрытия, считать текущие параметры окна, сериализовать их в JSON и записать на диск.

> **Важно.** В обработчике события `FormClosing` доступен аргумент `FormClosingEventArgs e`. Если в процессе сохранения настроек или проверки несохраненных данных возникнет необходимость отменить закрытие программы, можно установить свойство `e.Cancel = true;`.

Рассмотрим структуру данных, которую мы будем сохранять в файл `settings.json`. Для этого создадим специальный класс (или структуру) `AppSettings`:

```csharp
public class AppSettings
{
    public int WindowWidth { get; set; } = 800;
    public int WindowHeight { get; set; } = 600;
    public int WindowLeft { get; set; } = 100;
    public int WindowTop { get; set; } = 100;
    public FormWindowState WindowState { get; set; } = FormWindowState.Normal;
    public string LastOpenedFile { get; set; } = string.Empty;
    public bool DarkThemeEnabled { get; set; } = false;
}
```

Установка значений по умолчанию (например, `Width = 800`, `Height = 600`) гарантирует, что если файла конфигурации еще нет (первый запуск программы), приложение откроется в стандартном окне без сбоев и генерации исключений.

---

### Пример 1. Приложение, запоминающее размеры окна, позицию и последний открытый файл

Разработаем оконное приложение на Windows Forms, которое при запуске восстанавливает свое положение на экране, размеры, состояние (вернулось ли оно из развернутого на весь экран режима) и последний выбранный текстовый файл, а при закрытии сохраняет эти данные обратно в `settings.json`.

Полный листинг формы `MainForm.cs`:

```csharp
using System;
using System.IO;
using System.Text.Json;
using System.Windows.Forms;

namespace FileAppSettingsApp
{
    // Класс для хранения настроек приложения
    public class AppSettings
    {
        public int Width { get; set; } = 800;
        public int Height { get; set; } = 600;
        public int Left { get; set; } = 100;
        public int Top { get; set; } = 100;
        public int WindowStateValue { get; set; } = 0; // 0 - Normal, 1 - Maximized, 2 - Minimized
        public string LastFile { get; set; } = string.Empty;
    }

    public partial class MainForm : Form
    {
        private AppSettings _settings;
        private string _settingsFilePath;
        private TextBox textBoxContent;
        private MenuStrip menuStrip1;
        private ToolStripMenuItem fileToolStripMenuItem;
        private ToolStripMenuItem openToolStripMenuItem;

        public MainForm()
        {
            InitializeComponent();
            InitializeCustomControls();

            // Формируем путь к файлу настроек в AppData\Roaming\FileAppSettingsApp
            string roamingDir = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
            string appDir = Path.Combine(roamingDir, "FileAppSettingsApp");
            Directory.CreateDirectory(appDir);
            _settingsFilePath = Path.Combine(appDir, "settings.json");

            // Подписываемся на события загрузки и закрытия формы
            this.Load += MainForm_Load;
            this.FormClosing += MainForm_Closing;
        }

        // Программная инициализация элементов интерфейса для чистоты примера
        private void InitializeCustomControls()
        {
            this.Text = "Менеджер сессий и настроек";
            this.Width = 800;
            this.Height = 600;

            menuStrip1 = new MenuStrip();
            fileToolStripMenuItem = new ToolStripMenuItem("Файл");
            openToolStripMenuItem = new ToolStripMenuItem("Открыть...", null, OpenToolStripMenuItem_Click);

            fileToolStripMenuItem.DropDownItems.Add(openToolStripMenuItem);
            menuStrip1.Items.Add(fileToolStripMenuItem);
            this.MainMenuStrip = menuStrip1;
            this.Controls.Add(menuStrip1);

            textBoxContent = new TextBox();
            textBoxContent.Multiline = true;
            textBoxContent.Dock = DockStyle.Fill;
            textBoxContent.ScrollBars = ScrollBars.Vertical;
            this.Controls.Add(textBoxContent);
        }

        // Обработчик события Load: восстановление настроек
        private void MainForm_Load(object sender, EventArgs e)
        {
            try
            {
                if (File.Exists(_settingsFilePath))
                {
                    string jsonString = File.ReadAllText(_settingsFilePath);
                    // Десериализация JSON в объект настроек
                    _settings = JsonSerializer.Deserialize<AppSettings>(jsonString) ?? new AppSettings();
                }
                else
                {
                    _settings = new AppSettings();
                }

                // Применяем настройки к окну
                this.Width = _settings.Width;
                this.Height = _settings.Height;
                this.Left = _settings.Left;
                this.Top = _settings.Top;

                if (_settings.WindowStateValue == 1)
                {
                    this.WindowState = FormWindowState.Maximized;
                }
                else
                {
                    this.WindowState = FormWindowState.Normal;
                }

                // Если был сохранен путь к последнему открытому файлу, пробуем загрузить его
                if (!string.IsNullOrEmpty(_settings.LastFile) && File.Exists(_settings.LastFile))
                {
                    textBoxContent.Text = File.ReadAllText(_settings.LastFile);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Не удалось загрузить настройки: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                _settings = new AppSettings();
            }
        }

        // Обработчик события FormClosing: сохранение текущих настроек
        private void MainForm_Closing(object sender, FormClosingEventArgs e)
        {
            try
            {
                // Если окно свернуто или развернуто, сохраняем размеры нормального состояния
                if (this.WindowState == FormWindowState.Normal)
                {
                    _settings.Width = this.Width;
                    _settings.Height = this.Height;
                    _settings.Left = this.Left;
                    _settings.Top = this.Top;
                    _settings.WindowStateValue = 0;
                }
                else if (this.WindowState == FormWindowState.Maximized)
                {
                    _settings.WindowStateValue = 1;
                    // Размеры в Maximized не меняем, оставляем последние нормальные
                }

                // Сериализация объекта настроек в JSON строку с отступами (WriteIndented = true)
                var options = new JsonSerializerOptions { WriteIndented = true };
                string jsonString = JsonSerializer.Serialize(_settings, options);

                // Запись на диск
                File.WriteAllText(_settingsFilePath, jsonString);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Ошибка при сохранении настроек: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        // Обработчик пункта меню "Открыть"
        private void OpenToolStripMenuItem_Click(object sender, EventArgs e)
        {
            using (OpenFileDialog openFileDialog = new OpenFileDialog())
            {
                openFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
                if (openFileDialog.ShowDialog() == DialogResult.OK)
                {
                    _settings.LastFile = openFileDialog.FileName;
                    textBoxContent.Text = File.ReadAllText(_settings.LastFile);
                }
            }
        }
    }
}
```

### Пояснение по шагам работы примера 1:
1. **Инициализация путей:** В конструкторе формы вычисляется путь `AppData\Roaming\FileAppSettingsApp\settings.json`. Метод `Directory.CreateDirectory` гарантирует отсутствие ошибок, если папка запускается впервые.
2. **Загрузка (`Form_Load`):** Проверяется существование файла. Если файл есть, библиотека `System.Text.Json` считывает параметры в класс `AppSettings`. Окно принимает сохраненные координаты `Left`, `Top`, `Width`, `Height`. Если файл поврежден или отсутствует, программа не падает, а подставляет безопасные значения по умолчанию благодаря конструкции `?? new AppSettings()`.
3. **Сохранение (`Form_Closing`):** Перед самым выходом из приложения считываются актуальные свойства формы. С помощью `JsonSerializer.Serialize` данные упаковываются в текстовый JSON-формат и записываются на диск.

---

### Пример 2. Комплексное приложение «Заметки с категориями»

Создадим более сложное и полноценное приложение. Программа «Заметки с категориями» позволяет пользователю создавать текстовые заметки, распределять их по категориям, хранить список заметок в таблице `DataGridView` и автоматически сохранять всю базу данных в структурированный каталог документов пользователя при выходе, а также восстанавливать её при запуске.

В этом примере мы объединим сразу несколько изученных ранее тем: работу с таблицами `DataGridView`, бинарную или JSON-сериализацию коллекций данных (`BindingList<T>`), диалоговые окна и обработку исключений.

Структура данных одной заметки оформляется в виде класса:

```csharp
public class NoteItem
{
    public string Title { get; set; } = string.Empty;
    public string Category { get; set; } = "Общие";
    public string Content { get; set; } = string.Empty;
    public DateTime CreatedDate { get; set; } = DateTime.Now;
}
```

Ниже представлен полный код оконного приложения `CategoryNotesForm.cs`:

```csharp
using System;
using System.ComponentModel;
using System.IO;
using System.Text.Json;
using System.Windows.Forms;

namespace CategoryNotesApp
{
    public class NoteItem
    {
        public string Title { get; set; } = string.Empty;
        public string Category { get; set; } = "Общие";
        public string Content { get; set; } = string.Empty;
        public DateTime CreatedDate { get; set; } = DateTime.Now;
    }

    public partial class CategoryNotesForm : Form
    {
        private DataGridView dataGridViewNotes;
        private TextBox txtTitle;
        private ComboBox cmbCategory;
        private TextBox txtContent;
        private Button btnAdd;
        private Button btnDelete;
        
        // Коллекция с поддержкой уведомлений интерфейса
        private BindingList<NoteItem> notesList;
        private string dataFilePath;

        public CategoryNotesForm()
        {
            InitializeComponent();
            SetupLayout();

            // Путь к файлу данных в папке документов пользователя
            string docsFolder = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
            string appFolder = Path.Combine(docsFolder, "CategoryNotesManager");
            Directory.CreateDirectory(appFolder);
            dataFilePath = Path.Combine(appFolder, "notes_database.json");

            notesList = new BindingList<NoteItem>();
            dataGridViewNotes.DataSource = notesList;

            this.Load += CategoryNotesForm_Load;
            this.FormClosing += CategoryNotesForm_FormClosing;
        }

        private void SetupLayout()
        {
            this.Text = "Заметки с категориями и файловым хранилищем";
            this.Width = 900;
            this.Height = 650;

            // Таблица для отображения заметок
            dataGridViewNotes = new DataGridView();
            dataGridViewNotes.Location = new System.Drawing.Point(12, 12);
            dataGridViewNotes.Size = new System.Drawing.Size(400, 580);
            dataGridViewNotes.SelectionMode = DataGridViewSelectionMode.FullRowSelect;
            dataGridViewNotes.MultiSelect = false;
            dataGridViewNotes.AllowUserToAddRows = false;
            dataGridViewNotes.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill;
            this.Controls.Add(dataGridViewNotes);

            // Метки и поля ввода
            Label lbl1 = new Label() { Text = "Заголовок:", Location = new System.Drawing.Point(430, 12), AutoSize = true };
            txtTitle = new TextBox() { Location = new System.Drawing.Point(430, 30), Width = 430 };

            Label lbl2 = new Label() { Text = "Категория:", Location = new System.Drawing.Point(430, 65), AutoSize = true };
            cmbCategory = new ComboBox() { Location = new System.Drawing.Point(430, 85), Width = 430, DropDownStyle = ComboBoxStyle.DropDown };
            cmbCategory.Items.AddRange(new string[] { "Работа", "Учеба", "Личное", "Покупки", "Общие" });
            cmbCategory.Text = "Общие";

            Label lbl3 = new Label() { Text = "Текст заметки:", Location = new System.Drawing.Point(430, 120), AutoSize = true };
            txtContent = new TextBox() { Location = new System.Drawing.Point(430, 140), Width = 430, Height = 350, Multiline = true, ScrollBars = ScrollBars.Vertical };

            btnAdd = new Button() { Text = "Добавить заметку", Location = new System.Drawing.Point(430, 505, 200, 40) };
            btnAdd.Click += BtnAdd_Click;

            btnDelete = new Button() { Text = "Удалить выбранную", Location = new System.Drawing.Point(660, 505, 200, 40) };
            btnDelete.Click += BtnDelete_Click;

            this.Controls.AddRange(new Control[] { lbl1, txtTitle, lbl2, cmbCategory, lbl3, txtContent, btnAdd, btnDelete });
        }

        // Добавление новой заметки в список
        private void BtnAdd_Click(object sender, EventArgs e)
        {
            if (string.IsNullOrWhiteSpace(txtTitle.Text))
            {
                MessageBox.Show("Заполните заголовок заметки!", "Внимание", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                return;
            }

            NoteItem newItem = new NoteItem
            {
                Title = txtTitle.Text.Trim(),
                Category = cmbCategory.Text.Trim(),
                Content = txtContent.Text,
                CreatedDate = DateTime.Now
            };

            notesList.Add(newItem);

            // Очистка полей ввода
            txtTitle.Clear();
            txtContent.Clear();
            cmbCategory.Text = "Общие";
        }

        // Удаление выбранной заметки
        private void BtnDelete_Click(object sender, EventArgs e)
        {
            if (dataGridViewNotes.SelectedRows.Count > 0)
            {
                int index = dataGridViewNotes.SelectedRows[0].Index;
                notesList.RemoveAt(index);
            }
            else
            {
                MessageBox.Show("Выберите строку в таблице для удаления.", "Информация", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
        }

        // Загрузка базы данных при старте
        private void CategoryNotesForm_Load(object sender, EventArgs e)
        {
            try
            {
                if (File.Exists(dataFilePath))
                {
                    string json = File.ReadAllText(dataFilePath);
                    var loadedNotes = JsonSerializer.Deserialize<List<NoteItem>>(json);
                    
                    if (loadedNotes != null)
                    {
                        notesList.Clear();
                        foreach (var note in loadedNotes)
                        {
                            notesList.Add(note);
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Ошибка при загрузке базы данных заметок: {ex.Message}", "Ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        // Сохранение базы данных при закрытии формы
        private void CategoryNotesForm_FormClosing(object sender, FormClosingEventArgs e)
        {
            try
            {
                var options = new JsonSerializerOptions { WriteIndented = true };
                string json = JsonSerializer.Serialize(notesList, options);
                File.WriteAllText(dataFilePath, json);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Не удалось сохранить данные на диск: {ex.Message}", "Критическая ошибка", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }
}
```

### Разбор алгоритма и проверка отказоустойчивости:
1. **Связывание данных (`BindingList<T>`):** В отличие от обычного `List<T>`, использование `BindingList<NoteItem>` позволяет компоненту `DataGridView` автоматически обновлять отображение при добавлении или удалении элементов в коде.
2. **Хранение в документах:** База данных заметок сохраняется в каталог `Documents\CategoryNotesManager\notes_database.json`. Это гарантирует, что пользователь сможет легко найти файл, сделать его резервную копию или перенести на другой компьютер.
3. **Безопасность выполнения:** Весь код чтения и записи обернут в блоки `try-catch`. Если жесткий диск заполнится или у программы пропадут права на запись в папку документов, пользователь увидит понятное сообщение через `MessageBox`, а приложение не завершится аварийно с исключением необработанного сбоя.

## Типичные ошибки

1. **Забытый вызов `ShowDialog()` перед обращением к `FileName`.**
   * *Ошибка:* Разработчик вызывает `openFileDialog.FileName` сразу после создания объекта диалога, не проверив результат `ShowDialog()`.
   * *Почему возникает:* Непонимание модальности и асинхронности взаимодействия с пользователем; в результате переменная пути остается пустой, что вызывает логическую ошибку.
   * *Как избежать:* Всегда оборачивать операции с файлом в проверку `if (openFileDialog.ShowDialog() == DialogResult.OK)`.

2. **Нарушение порядка чтения/записи в бинарных потоках (`BinaryReader` / `BinaryWriter`).**
   * *Ошибка:* Записали в файл сначала `int`, а затем `string`, а читают в обратном порядке (`ReadString()`, затем `ReadInt32()`).
   * *Почему возникает:* Отсутствие строгой спецификации структуры двоичного файла.
   * *Как избежать:* Четко документировать порядок полей. Помните главное правило: порядок чтения методов `BinaryReader` должен в точности повторять порядок вызовов `BinaryWriter`.

3. **Отсутствие обработки исключений `IOException` и `UnauthorizedAccessException`.**
   * *Ошибка:* Программа вылетает с аварийным сообщением при попытке открыть занятый другим процессом файл или при сохранении в защищенную системную папку (например, `C:\Program Files`).
   * *Почему возникает:* Игнорирование особенностей файловой системы ОС Windows.
   * *Как избежать:* Всегда помещайте код работы с потоками внутрь блоков `try-catch`.

4. **Забытый блок `using` для файловых потоков.**
   * *Ошибка:* Создается `StreamReader` или `FileStream`, но метод `.Dispose()` или `Close()` не вызывается.
   * *Почему возникает:* Программист полагается на то, что сборщик мусора сам все очистит. Но неуправляемые дескрипторы ОС не освобождаются мгновенно, что приводит к блокировке файла (`FileLock`).
   * *Как избежать:* Использовать конструкцию `using` (или декларацию `using` в современных версиях C#).

5. **Путаница с кодировками текста.**
   * *Ошибка:* Файл сохраняется в кодировке по умолчанию (`Encoding.Default`), а открывается в `UTF-8` (или наоборот), из-за чего русские буквы превращаются в «кракозябры».
   * *Почему возникает:* Неявное создание объектов `StreamReader` без указания параметра `Encoding`.
   * *Как избежать:* Всегда явно передавать `Encoding.UTF8` в конструкторы `StreamReader` и `StreamWriter`.

6. **Потеря изменений при закрытии формы или открытии нового файла.**
   * *Ошибка:* Приложение закрывается или открывает другой документ, молча стирая текущие отредактированные данные в `TextBox` или `ListBox`.
   * *Почему возникает:* Отсутствие флага модификации (`isDirty`) и проверки при закрытии формы (`FormClosing`).
   * *Как избежать:* Реализовать переменную состояния изменений и метод подтверждения сохранения перед сбросом данных.

7. **Некорректная индексация или синтаксис строки свойства `Filter`.**
   * *Ошибка:* Написание фильтра без разделителя `|` или с ошибкой в маске (например, `"Text files *.txt"` вместо `"Текст (*.txt)|*.txt"`).
   * *Почему возникает:* Незнание строгого синтаксиса строки фильтрации диалоговых окон Windows.
   * *Как избежать:* Проверять работу фильтра запуском и сверяться со стандартами синтаксиса `Описание|маска|Описание2|маска2`.

## Вопросы для самопроверки

1. Какой базовый класс объединяет все стандартные диалоговые окна файловой системы в пространстве имен `System.Windows.Forms`?
2. Какое значение возвращает метод `ShowDialog()` у диалогового окна, если пользователь нажал кнопку подтверждения выбора («Открыть» или «Сохранить»)?
3. В чем разница между свойствами `FileName` и `FileNames` у компонента `OpenFileDialog`?
4. Для чего предназначено свойство `Filter` в диалоговых окнах и каков синтаксис разделения описания и маски?
5. Почему использование абсолютных жестко закодированных путей вроде `"C:\\data\\file.txt"` считается грубой ошибкой проектирования?
6. Какую роль выполняет конструкция `using` при работе с потоками `StreamReader` и `StreamWriter`?
7. Почему при сохранении списка из `ListBox` в текстовый файл рекомендуется использовать метод `WriteLine`, а не `Write`?
8. Как связаны между собой классы `BinaryWriter`, `BinaryReader` и базовый класс потока `Stream`?
9. Почему порядок вызова методов чтения `BinaryReader` должен строго соответствовать порядку вызовов `BinaryWriter` при записи?
10. Что делает коллекция `BindingList<T>` при привязке ее к элементу управления `DataGridView` и почему она удобнее ручного добавления строк?
11. Зачем при чтении бинарного файла используется проверка `reader.BaseStream.Position < reader.BaseStream.Length`?
12. Какое исключение генерирует операционная система, если приложение пытается записать файл в директорию, на которую у пользователя нет административных прав?

## Задачи для самостоятельного решения

1. **Задача 1 (Диалоговое окно и фильтры).** Разработать оконное приложение с кнопкой «Выбрать лог-файл». При нажатии должен открываться `OpenFileDialog`, настроенный только на файлы с расширением `.log` и все файлы `*.*`. Если пользователь выбрал файл и нажал ОК, в `MessageBox` должен вывестись полный путь к файлу и его имя, извлеченное с помощью статического класса `Path`.
2. **Задача 2 (Счетчик слов в текстовом файле).** Написать программу, которая открывает текстовый документ через `OpenFileDialog`, считывает его содержимое с помощью `StreamReader` в кодировке `UTF8`, выводит текст в `TextBox`, а в заголовке формы отображает общее количество символов и количество строк в прочитанном файле.
3. **Задача 3 (Экспорт `ComboBox` в файл).** На форме расположен `ComboBox`, содержащий 5 названий городов (статически добавленных в дизайнере или коде), кнопка «Экспортировать». По нажатию кнопки открывается `SaveFileDialog` с расширением по умолчанию `.txt`. Каждый город из выпадающего списка должен быть записан на отдельной строке текстового файла.
4. **Задача 4 (Резервное копирование по расписанию/кнопке).** Создать приложение с кнопкой «Создать резервную копию». При нажатии открывается `OpenFileDialog` для выбора любого текстового файла. Программа должна прочитать этот файл и создать его копию в той же папке, но с добавлением суффикса `_backup.txt` к имени файла (используйте методы класса `Path`).
5. **Задача 5 (Бинарная структура координат).** Спроектировать структуру `PointData` с полями `double X`, `double Y` и `string Label`. Создать массив из 3 таких точек, записать их в бинарный файл с помощью `BinaryWriter`. Затем написать функцию чтения этого файла с помощью `BinaryReader`, которая восстанавливает массив и выводит значения в консоль отладки (`System.Diagnostics.Debug.WriteLine`).
6. **Задача 6 (Таблица сотрудников в `DataGridView`).** Создать форму с `DataGridView`, кнопкой «Сохранить в бинарный файл» и кнопкой «Загрузить из бинарного файла». Описать структуру `Employee` с полями `string FullName` (ФИО), `int Age` (возраст) и `double Salary` (зарплата). Привязать коллекцию `BindingList<Employee>` к таблице. Реализовать сохранение всей таблицы в бинарный формат и ее последующее восстановление с проверкой целостности потока.
7. **Задача 7 (Контроль изменений в блокноте).** Разработать мини-редактор текста с многострочным `TextBox`. Реализовать логику флага модификации (`isDirty`). Если текст изменяется в событии `TextChanged`, в шапке формы появляется символ `*`. При закрытии формы, если текст не сохранен, приложение выводит `MessageBox` с тремя кнопками (Да, Нет, Отмена). При нажатии «Да» происходит сохранение, при «Нет» форма закрывается без сохранения, при «Отмена» закрытие формы отменяется.

## Ответы к задачам

1. При успешном выборе файла на экране появляется модальное окно `MessageBox`, содержащее абсолютный путь к файлу (свойство `FileName`) и его имя с расширением, извлеченное методом `Path.GetFileName`. При отмене диалогового окна никаких действий не происходит.
2. Текст открытого файла полностью отображается в многострочном `TextBox`. В заголовке формы отображается строка в формате `ИмяФайла.txt | Символов: N | Строк: M`, где `N` — общее количество символов в прочитанной строке, а `M` — количество строк в тексте.
3. В результате работы программы и выбора пользователем пути через `SaveFileDialog` на диске создается текстовый файл, содержащий ровно 5 строк с названиями городов из выпадающего списка `ComboBox`, записанных последовательно в кодировке UTF-8.
4. В директории исходного выбранного файла создается его точная копия с тем же расширением, но с добавлением суффикса `_backup` к базовому имени файла (например, `document_backup.txt`).
5. Данные успешно сериализуются в бинарный поток с помощью `BinaryWriter` и десериализуются обратно с помощью `BinaryReader`. Массив из 3 точек корректно восстанавливается в памяти, а их числовые координаты и строковые метки без искажений выводятся в отладочный вывод среды разработки.
6. Коллекция `BindingList<Employee>`, привязанная к `DataGridView`, полностью сериализуется в бинарный файл (с предварительной записью количества элементов для контроля целостности потока) и корректно восстанавливается обратно в таблицу при загрузке.
7. При изменении текста в `TextBox` заголовок формы динамически помечается символом `*`. Попытка закрыть приложение при наличии несохраненных изменений вызывает модальный диалог с кнопками «Да», «Нет» и «Отмена»: выбор «Да» и «Нет» разрешает закрытие (с предварительным сохранением в первом случае), а выбор «Отмена» перехватывает событие закрытия формы и оставляет приложение запущенным.
