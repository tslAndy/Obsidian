#smells 
- Use a Guard Clause
- Extract Conditional
- Replace with Polymorphism
- Use Strategy Pattern
- Use Null Object
- Use Functional Programming Based Solution

```python
class Exporter:
    def export(self, export_format: str):
        if export_format == 'wav':
            self.exportInWav()
        elif export_format == 'flac':
            self.exportInFlac()
        elif export_format == 'mp3':
            self.exportInMp3()
        elif export_format == 'ogg':
            self.exportInOgg()

#----------------------------------------------

class Exporter:
    def export(self, export_format: str):
        exporter = self.get_format_factory(export_format)
        exporter.export()
        
    def get_format_factory(self, export_format: str):
        if export_format in self.export_format_factories:
            return render_factory[export_format]
        raise MissingFormatException
            ...
```