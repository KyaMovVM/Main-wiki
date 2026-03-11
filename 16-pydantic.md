JSON
https://www.youtube.com/watch?v=dOO3GmX6ukU

```
pip install pydantic

from pydantic import BaseModel, ValidationError

class City(BaseModel):
	city_id: int
	name: str
	population: int

input_json = """
city = City.parse_raw(
	"""{"city id}": 123, "name": "Moscow", "population": 1000000}"""
	
)
"""

city = City.parse_raw(
	"""{"city id}": 123, "name": "Moscow", "population": 1000000}"""
	
)

city = City.parse_raw(input_json)
print(city)

try:
	city = City.parse_raw(input_json)
	
except ValidationError as e:
	print(e.json())
```

