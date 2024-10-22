#smells
- [[Extract Method]]
- [[Extract Variable]]
```python
regex_pattern = '(\W|^)(\w*)\s-\s[0-9]?[0-9]:[0-9][0-9]'

#---------------------------------------------------

def get_regex_pattern_current_city_time() -> str:
    """
    Example Match:
      - `Wroclaw - 17:42`
      - `Berlin - 17:42`
      - `San Jose - 10:42`

    Compiled: (\W|^)(\w*)\s-\s[0-9]?[0-9]:[0-9][0-9]
    """
    prevent_excessive_match = '(\W|^)'
    city = '(\w*)'
    indication = '\s-\s'
    hour = '[0-9]?[0-9]'
    minute = '[0-9][0-9]'
    time = f'{hour}:{minute}'
    return f"{prevent_excessive_match}{city}{indication}{time}"
```