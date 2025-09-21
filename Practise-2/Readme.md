# Читання файлу
```csv 
ім'я
"Іван Петро"
"Олена Шевченко"
"Петро Іваненко"
```
## Вілкритання файлу
``` С++ 
 ifstream file(path);
    if (!file.is_open()) {
        cerr << "Could not open file " << path << endl;
        return false;
    }
```
## Читання файлу 
``` С++ 
	vector<string> user_names(3);
	....
	string line;
    getline(file, line); // пропускаємо перший рядок
 	while (getline(file, line)) {
        constexpr char SEPARATOR = ',';
        if (line.empty()) {
            continue;

        stringstream row(line); // читаємо весь рядок
		string name; // створюємо тимчасову змінну 

 		std::getline(row, name, SEPARATOR); // читаємо перший елемент та записуємо у тимчасову зміну

		... // Додаткова логіка
		
		 user_names.push_back(name);
 	}	
```