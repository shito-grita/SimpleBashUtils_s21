#include <ctype.h>
#include <regex.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_LINE_LENGTH 1024
#define MAX_PATTERN_LENGTH 256
#define INITIAL_PATTERN_CAPACITY 10

typedef struct {
  char **pattern;
  int pattern_count;
  int show_filename;      // Флаг -h:
  int only_matches;       // Флаг -o:
  int invert_match;       // Флаг -v:
  int count_only;         // Флаг -c:
  int list_files;         // Флаг -l:
  int show_line_numbers;  // Флаг -n:
  int ignore_case;        // Флаг -i:
  int silent;             // Флаг -s:
} GrepOptions;

void print_only_matches(const char *line, const GrepOptions *options) {
  for (int i = 0; i < options->pattern_count; i++) {
    regex_t regex;
    regcomp(&regex, options->pattern[i],
            REG_EXTENDED | (options->ignore_case ? REG_ICASE : 0));

    const char *match_start = line;
    while ((match_start = strstr(match_start, options->pattern[i])) != NULL) {
      printf("%s\n", options->pattern[i]);
      match_start += strlen(options->pattern[i]);
    }

    regfree(&regex);
  }
}
void count_matches(int *total_matches, const char *line,
                   const GrepOptions *options) {
  for (int i = 0; i < options->pattern_count; i++) {
    regex_t regex;
    regcomp(&regex, options->pattern[i],
            REG_EXTENDED | (options->ignore_case ? REG_ICASE : 0));

    if (!regexec(&regex, line, 0, NULL, 0)) {
      (*total_matches)++;
    }
    regfree(&regex);
  }
}

void grep(const GrepOptions *options, const char *filename) {
  FILE *file = fopen(filename, "r");
  if (file == NULL) {
    return;
  }

  char line[MAX_LINE_LENGTH];
  int line_number = 0;
  int total_matches = 0;

  while (fgets(line, sizeof(line), file)) {
    line_number++;
    int match_found = 0;

    for (int i = 0; i < options->pattern_count; i++) {
      regex_t regex;
      regcomp(&regex, options->pattern[i],
              REG_EXTENDED | (options->ignore_case ? REG_ICASE : 0));
      if (!regexec(&regex, line, 0, NULL, 0)) {
        match_found = 1;
        break;
      }
      regfree(&regex);
    }

    if ((match_found && !options->invert_match) ||
        (!match_found && options->invert_match)) {
      if (options->count_only) {
        count_matches(&total_matches, line, options);
      } else {
        if (options->show_filename) {
          printf("%s:", filename);
        }
        if (options->show_line_numbers) {
          printf("%d: ", line_number);
        }
        if (options->only_matches) {
          print_only_matches(line, options);
        } else {
          printf("%s", line);
        }
      }
    }
  }

  if (options->count_only) {
    printf("%d\n", total_matches);
  }

  fclose(file);
}

void parse_arguments(int argc, char *argv[], GrepOptions *options) {
  options->show_filename = 1;  // По умолчанию показывать имя файла
  options->only_matches = 0;  // По умолчанию не выводить только совпадения
  options->invert_match = 0;  // По умолчанию не инвертировать поиск
  options->count_only =
      0;  // По умолчанию не выводить только количество совпадений
  options->list_files = 0;  // По умолчанию не выводить имена файлов
  options->show_line_numbers = 0;  // По умолчанию не показывать номера строк
  options->pattern_count = 0;
  options->ignore_case = 0;
  options->silent = 0;

  options->pattern = malloc(INITIAL_PATTERN_CAPACITY * sizeof(char *));
  if (!options->pattern) {
    exit(EXIT_FAILURE);
  }
  int pattern_capacity = INITIAL_PATTERN_CAPACITY;
  for (int i = 1; i < argc; i++) {
    if (strcmp(argv[i], "-h") == 0) {
      options->show_filename = 0;
    } else if (strcmp(argv[i], "-o") == 0) {
      options->only_matches = 1;
    } else if (strcmp(argv[i], "-I") == 0) {
      continue;
    } else if (strcmp(argv[i], "-v") == 0) {
      options->invert_match = 1;
    } else if (strcmp(argv[i], "-c") == 0) {
      options->count_only = 1;
    } else if (strcmp(argv[i], "-l") == 0) {
      options->list_files = 1;
      options->show_filename = 0;
    } else if (strcmp(argv[i], "-n") == 0) {
      options->show_line_numbers = 1;
    } else if (strcmp(argv[i], "-i") == 0) {
      options->ignore_case = 1;
    } else if (strcmp(argv[i], "-s") == 0) {
      options->silent = 1;
    } else if (strcmp(argv[i], "-e") == 0 && i + 1 < argc) {
      i++;
      if (options->pattern_count >= pattern_capacity) {
        pattern_capacity *= 2;
        options->pattern =
            realloc(options->pattern, pattern_capacity * sizeof(char *));
        if (!options->pattern) {
          exit(EXIT_FAILURE);
        }
      }
      options->pattern[options->pattern_count] =
          malloc((strlen(argv[i]) + 1) * sizeof(char));
      if (!options->pattern[options->pattern_count]) {
        exit(EXIT_FAILURE);
      }
      strcpy(options->pattern[options->pattern_count++], argv[i]);
    } else {
      if (options->pattern_count >= pattern_capacity) {
        pattern_capacity *= 2;
        options->pattern =
            realloc(options->pattern, pattern_capacity * sizeof(char *));
        if (!options->pattern) {
          exit(EXIT_FAILURE);
        }
      }
      options->pattern[options->pattern_count] =
          malloc((strlen(argv[i]) + 1) * sizeof(char));
      if (!options->pattern[options->pattern_count]) {
        exit(EXIT_FAILURE);
      }
      strcpy(options->pattern[options->pattern_count++], argv[i]);
    }
  }
}

int main(int argc, char *argv[]) {
  GrepOptions options;
  parse_arguments(argc, argv, &options);

  if (options.pattern_count == 0 || argc <= options.pattern_count) {
    fprintf(stderr, "Использование: %s [опции] <шаблон> [<шаблон>...] <файл>\n",
            argv[0]);
    return EXIT_FAILURE;
  }
  const char *filename = argv[argc - 1];
  grep(&options, filename);

  for (int i = 0; i < options.pattern_count; i++) {
    free(options.pattern[i]);
  }
  free(options.pattern);
  return EXIT_SUCCESS;
}

