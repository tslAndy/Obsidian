Управление ресурсами на основе области действия — очень распространенная и важная техника управления ресурсами, где ресурс привязан к жизненному циклу объекта. Это делается путем выделения ресурса в конструкторе вспомогательного объекта и освобождения ресурса в деструкторе вспомогательного объекта.

Допустим, вы написали функцию, которая захватывает [мьютекс](http://en.wikipedia.org/wiki/Mutual_exclusion) в начале и освобождает его в конце:

```cs
void fun(Mutex& mutex)
{
  mutex.Acquire();
 
  // do stuff here
 
  mutex.Release();
}
```

Позже вы захотите добавить early-out в середине функции. Поскольку функция не слишком длинная, вы быстро заметите, что функция получает мьютекс в начале и освобождает его в конце, поэтому вы не забываете освободить мьютекс перед early-out:

```cs
void fun(Mutex& mutex)
{
  mutex.Acquire();
 
  // do stuff here
 
  if (earlyOut)
  {
    // whew, good thing I remember to do this
    mutex.Release();
    return;
  }
 
  // do stuff here
 
  mutex.Release();
}
```

Теперь прошел год, и функция превратилась в чудовище из тонн строк. Было добавлено еще много ранних выходов, и вы всегда помните, что нужно освободить мьютекс перед всеми ранними выходами. Однажды кто-то еще придет и решит добавить еще один ранний выход в функцию. Он может не заметить захват и освобождение мьютекса, зарытые в море кода:

```cs
void fun(Mutex &mutex)
{
  mutex.Acquire();
 
  // many lines of code
 
  if (newEarlyOut)
  {
    // oops...
    return;
  }
 
  // many lines of code
 
  mutex.Release();
}
```

Ручное управление ресурсами обычно подвержено именно этому типу человеческих ошибок. Управление ресурсами на основе области действия является идеальным решением этой проблемы. В стандартах C++ объект в стеке всегда будет выполнять свой конструктор при инициализации и выполнять свой деструктор, когда программа покидает свою область действия. Мы можем использовать это в своих интересах для автоматизации управления ресурсами. Вот наш вспомогательный класс для автоматического получения и освобождения мьютекса:

```cs
class AutoMutexLock
{
  public:
 
    AutoMutexLock(Mutex &mutex)
      : m_mutex(mutex)
    {
      m_mutex.Acquire();
    }
 
    ~AutoMutexLock(void)
    {
      m_mutex.Release();
    }
 
  private:
     
    Mutex &m_mutex;
}
```
Теперь нам нужно только получить мьютекс в начале функции, и нам больше не нужно помнить о необходимости освобождать мьютекс перед каждым возвратом функции, потому что деструктор вспомогательного класса сделает это за нас:
```cs
void fun(Mutex &mutex)
{
  AutoMutexLock lock(mutex);
 
  // many lines of code
 
  if (newEarlyOut)
  {
    // no need to release mutex here
    return;
  }
 
  // many lines of code
 
  if (earlyOut)
  {
    // nor here
    return;
  }
 
  // many lines of code
 
  // not at the end, either
}
```

Этот метод может быть распространен на множество сценариев. Например, если вы выделяете память из кучи в функции и хотите, чтобы память освобождалась при выходе программы из функции, вы можете использовать интеллектуальные [указатели](http://en.wikipedia.org/wiki/Smart_pointer) в [STL](http://en.wikipedia.org/wiki/Standard_Template_Library) . Следующий пример демонстрирует простую реализацию класса интеллектуального указателя:
```cs
template <typename T>
class SmartPtr
{
  public:
 
    SmartPtr(T *ptr)
      : m_ptr(ptr)
    { }
 
    ~SmartPtr(void)
    {
      delete m_ptr;
    }
 
    T &operator*(void)
    {
      return *m_ptr;
    }
 
    T *operator->(void)
    {
      return m_ptr;
    }
 
  private:
 
    T *m_ptr;
}
```
А клиентский код может выглядеть примерно так:
```cs
void fun(void)
{
  // allocate memory here
  SmartPointer ptr(new MyDataClass());
 
  // many lines of code
   
  // this works because operator-> is properly overloaded
  ptr->DoStuff();
   
  if (earlyOut)
  {
    // memory automatically freed here
    return;
  }
 
  // many lines of code
 
  // memory automatically freed here, too
}
```

Вот еще один пример, связанный с памятью. Разработчики консольных игр очень часто заранее выделяют всю доступную память на консоли при запуске игры. Некоторые разделяют ее на фрагменты, идентифицируемые «контекстами памяти». Допустим, у вас есть глобальный стек контекстов памяти, где верхний контекст памяти определяет, на какой фрагмент предварительно выделенной памяти будет указывать указатель, возвращаемый оператором `new`.

Если вы хотите использовать определенный кусок предварительно выделенной памяти в области действия функции, вы можете поместить соответствующий контекст памяти в стек в начале функции, а затем вытащить его, когда программа выходит из функции. Это должно показаться вам ужасно знакомой задачей. Вот наш вспомогательный класс:
```cs
class AutoMemoryContext
{
  public:
 
    AutoMemoryContext(MemoryContext &context)
      : m_context(context)
    {
      PushMemoryContxt(context);
    }
 
    ~AutoMemoryContext(void)
    {
      PopMemoryContext(context);
    }
 
  private:
 
    MemoryContext &m_context;
}
```

И клиентский код не представляет собой ничего удивительного:

```cs
void fun(void)
{
  // push memory context here
  AutoMemoryContext autoContext(kAiMemoryContext);
 
  // many lines of code
   
  if (earlyOut)
  {
    // memory context will be automatically popped here
    return;
  }
 
  // many lines of code
 
  // memory context will be automatically popped here, too
}
```

Благодаря управлению ресурсами на основе области действия вы практически бесплатно получаете логику освобождения ресурса, правильно вставленного компилятором!