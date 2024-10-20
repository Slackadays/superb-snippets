# ✂️ Welcome to Superb Snippets

Super Snippets is my collection of C++ snippets that solve common tasks most programs of yours probably need to do but for which lack an obvious and/or optimized solution. 

To use these snippets, just copy and paste them into your own code. And if they don't fit your needs perfectly, feel free to customize them.

Superb Snippets are better than whatever AI you use for coding because these have been verified to work, and work well they do!

## Where these came from

Most of these snippets are from my projects like [Clipboard](https://github.com/Slackadays/Clipboard), so they're already optimized, tested, and used in a production setting.

If these snippets change upstream, then I may update them here.

## Licensing?

Although I pulled these snippets from my projects which have licenses like the GPL, all the snippets here fall under The Unlicense which means you can use, modify, and redistribute them without attribution or other requirements or limitations, because I wrote them myself. Why? I did this to make it super duper easy to hit the ground running when it comes to the tasks these snippets solve!

# The Snippets

## Are we running on a Unix-like (Linux, Haiku) or Unix (BSD, macOS) system?

```cpp
#if defined(__linux__) || defined(__unix__) || defined(__APPLE__) || defined(__NetBSD__) || defined(__OpenBSD__) || defined(__DragonFly__) || defined(__HAIKU__) || defined(__FreeBSD__) \
        || defined(__posix__)
#define UNIX_OR_UNIX_LIKE
#endif
```

This checks if we're running on a variety of systems considered Unix-like or downright Unix.

### Requirements

None

### Example

```cpp
#if defined(UNIX_OR_UNIX_LIKE)
std::cout << "Not running on Windows!" << std::endl;
#endif
```

## Just get everything contained in a file

```cpp
std::optional<std::string> fileContents(const fs::path& path) {
#if defined(UNIX_OR_UNIX_LIKE)
    errno = 0;
    int fd = open(path.string().data(), O_RDONLY);
    if (fd == -1) {
        if (errno == ENOENT)
            return std::nullopt;
        else
            throw std::runtime_error("Couldn't open file " + path.string() + ": " + std::strerror(errno));
    }
    std::string contents;
#if defined(__linux__) || defined(__FreeBSD__)
    std::array<char, 65536> buffer;
#elif defined(__APPLE__) || defined(__OpenBSD__) || defined(__NetBSD__) || defined(__DragonFly__)
    std::array<char, 16384> buffer;
#else
    std::array<char, PIPE_BUF> buffer;
#endif
    ssize_t bytes_read;
    errno = 0;
    while ((bytes_read = read(fd, buffer.data(), buffer.size())) > 0) {
        contents.append(buffer.data(), bytes_read);
        if (bytes_read < buffer.size() && errno == 0) break; // check if we reached EOF early and not due to an error
    }
    close(fd);
    return contents;
#else
    std::stringstream buffer;
    std::ifstream file(path, std::ios::binary);
    if (!file.is_open()) return std::nullopt;
    buffer << file.rdbuf();
    return buffer.str();
#endif
}
```

This reads everything from a file if it exists and returns its raw content in a `std::optional<std::string>`. If the file doesn't exist, this returns a blank value provided by `std::optional`.

### Requirements

`<optional>`, `<array>`, `<string>`, `<filesystem>`, and `<unistd.h>` on Unix or Unix-like platforms, or `<fstream>` on Windows.

### Example

```cpp
auto content = fileContents("foobar.txt");
if (content.has_value())
  std::cout << content << std::endl;
else
  std::cout << "This file does not exist" << std::endl;
```

## Just write something to a file

```cpp
size_t writeToFile(const fs::path& path, const std::string& content, bool append = false) {
    std::ofstream file(path, append ? std::ios::app : std::ios::trunc | std::ios::binary);
    file << content;
    return content.size();
}
```

This writes arbitrary content to a specified file and returns how many bytes it wrote.

### Requirements

`<filesystem>`, `<string>`, `<fstream>`

### Example

```cpp
auto content = "Your mother";
writeToFile("Gigantic_Objects.txt", content);
```

## Is an environment variable set to a "true" value?

```cpp
bool envVarIsTrue(const std::string_view& name) {
    auto temp = getenv(name.data());
    if (temp == nullptr) return false;
    std::string result(temp);
    std::transform(result.begin(), result.end(), result.begin(), [](unsigned char c) { return std::tolower(c); });
    if (result == "1" || result == "true" || result == "yes" || result == "y" || result == "on" || result == "enabled") return true;
    return false;
}
```

This checks if an environment variable is set to a value that you can interpret as "true."

### Requirements

`<string_view>`, `<string>`

### Example

```cpp
bool skip_some_process = envVarIsTrue("SKIP_SOME_PROCESS");
std::cout << skip_some_process << std::endl;
```

## Find Levenshtein distance between any two strings

```cpp
size_t levenshteinDistance(const std::string_view& one, const std::string_view& two) {
    if (one == two) return 0;

    if (one.empty()) return two.size();
    if (two.empty()) return one.size();

    std::vector<std::vector<size_t>> matrix(one.size() + 1, std::vector<size_t>(two.size() + 1));

    for (size_t i = 0; i <= one.size(); i++)
        matrix.at(i).at(0) = i;

    for (size_t j = 0; j <= two.size(); j++)
        matrix.at(0).at(j) = j;

    for (size_t i = 1; i <= one.size(); i++) {
        for (size_t j = 1; j <= two.size(); j++) {
            if (one.at(i - 1) == two.at(j - 1))
                matrix.at(i).at(j) = matrix.at(i - 1).at(j - 1);
            else
                matrix.at(i).at(j) = std::min({matrix.at(i - 1).at(j - 1), matrix.at(i - 1).at(j), matrix.at(i).at(j - 1)}) + 1;
        }
    }

    return matrix.at(one.size()).at(two.size());
};
```

This calculates the Levenshtein distance between two strings using any character as a valid difference.

### Requirements

`<string_view>`, `<vector>`

### Example

```cpp
auto difference = levenshteinDistance("hello", "hallo");
// difference = 1
```
