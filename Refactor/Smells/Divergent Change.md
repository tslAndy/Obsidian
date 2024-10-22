#smells 
Если добавление простой функции заставляет разработчика изменить много, казалось бы, не связанных методов внутри класса, это указывает на запах кода _Divergent Change_.

Разница между *Divergent Change* и [[Shotgun Surgery]]  заключается в том, что *Divergent Change* решает проблему в рамках класса, а *Shotgun Surgery* — между классами.

- [[Extract Superclass]], [[Extract Subclass]] or new class
- [[Extract Method]]
- [[Move Method]]

```python
class ReportModifier:
    def get_report(self, report_name):
        ...
        return report

    def modify_report(self, report, new_entry):
        ...
        return modified_report

    def run(self, report_name, new_entry):
        report = self.get_report(report_name)
        return self.modify_report(report, new_entry)


report_modifier = ReportModifier(...)
modified_report = report_modifier.run('raport.csv', 'Parsed')

#----------------------------------------------------

class ReportReader:
    def get_report(self, report_name):
        ...
        return report

class ReportModifier:
    def modify_report(self, report, new_entry):
        ...
        return modified_report

report_reader = ReportReader(...)
report = report_reader.get_report('raport.csv')

report_modifier = ReportModifier(...)
modified_report = report_modifier.modify_report(report, 'Parsed')
```
