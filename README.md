# ExamRank03

## ft_printf

<details>
  <summary>subject</summary>

	Assignment name  : ft_printf
	Expected files   : ft_printf.c
	Allowed functions: malloc, free, write, va_start, va_arg, va_copy, va_end
	--------------------------------------------------------------------------------

	Write a function named `ft_printf` that will mimic the real printf but 
	it will manage only the following conversions: s,d and x.

	Your function must be declared as follows:

	int ft_printf(const char *, ... );

	Before you start we advise you to read the `man 3 printf` and the `man va_arg`.
	To test your program compare your results with the true printf.

	Exemples of the function output:

	call: ft_printf("%s\n", "toto");
	out: toto$

	call: ft_printf("Magic %s is %d", "number", 42);
	out: Magic number is 42%

	call: ft_printf("Hexadecimal for %d is %x\n", 42, 42);
	out: Hexadecimal for 42 is 2a$

	Obs: Your function must not have memory leaks. Moulinette will test that.

</details>

```c
#include <limits.h>
#include <stdarg.h>
#include <unistd.h>

typedef struct s_sc
{
	int len;
	int widht;
} t_sc;

void ft_putchar(char c)
{
	write(1, &c, 1);
}

void ft_putstr(char *s)
{
	int i;

	i = 0;
	while (s[i])
	{
		ft_putchar(s[i]);
		i++;
	}
}

int ft_strlen(const char *s)
{
	int i;

	i = 0;
	while (s[i])
		i++;
	return (i);
}

char *ft_strchr(const char *s)
{
	while (*s)
	{
		if (*s == '%')
			return ((char *)s);
		s++;
	}
	if (!s)
		return ((char *)s);
	return (NULL);
}

void ft_putnbr(int nb)
{
	if (nb == INT_MIN)
	{
		write(1, "-2147483648", 11);
		return;
	}
	if (nb >= 0 && nb <= 9)
		ft_putchar(nb + 48);
	else if (nb < 0)
	{
		ft_putchar('-');
		ft_putnbr(-nb);
	}
	else
	{
		ft_putnbr(nb / 10);
		ft_putnbr(nb % 10);
	}
}

int ft_intlen(int nb, char c)
{
	int i;
	int number;
	int neg;

	i = 0;
	if (!nb)
		return (1);
	if (nb < 0)
	{
		neg = 1;
		number = -nb;
	}
	else
	{
		neg = 0;
		number = nb;
	}
	if (c == 'd')
	{
		while (number)
		{
			number /= 10;
			i++;
		}
		return (i + neg);
	}
	return (0);
}

void ft_printhexa(unsigned int x, t_sc *sc)
{
	char *hexa;
	int res[100] = {0};
	int i;

	hexa = "0123456789abcdef";
	i = 0;
	while (x >= 16)
	{
		res[i] = hexa[x % 16];
		x = x / 16;
		i++;
	}
	res[i] = hexa[x];
	sc->len += i + 1;
	while (i >= 0)
	{
		ft_putchar(res[i]);
		i--;
	}
}

const char *ft_search_arg(va_list arg, const char *format, t_sc *sc)
{
	int d;
	char *s;
	unsigned int x;

	if (*format == 'd')
	{
		d = va_arg(arg, int);
		ft_putnbr(d);
		sc->len += ft_intlen(d, *format);
	}
	else if (*format == 's')
	{
		s = va_arg(arg, char *);
		if (!s)
		{
			write(1, "(null)", 6);
			sc->len += 6;
		}
		else
		{
			ft_putstr(s);
			sc->len += ft_strlen(s);
		}
	}
	else if (*format == 'x')
	{
		x = va_arg(arg, unsigned int);
		ft_printhexa(x, sc);
	}
	else
		return (NULL);
	format++;
	return (format);
}

const char *ft_read_text(t_sc *sc, const char *format)
{
	char *next;

	next = ft_strchr(format);
	if (next)
		sc->widht = next - format;
	else
		sc->widht = ft_strlen(format);
	write(1, format, sc->widht);
	sc->len += sc->widht;
	while (*format && *format != '%')
		++format;
	return (format);
}

int ft_printf(const char *format, ...)
{
	va_list arg;
	t_sc sc;

	va_start(arg, format);
	sc.len = 0;
	sc.widht = 0;
	while (*format)
	{
		if (*format == '%')
			format = ft_search_arg(arg, format + 1, &sc);
		else
			format = ft_read_text(&sc, format);
		if (!format)
		{
			write(1, "(null)", 6);
			va_end(arg);
			return (sc.len);
		}
	}
	va_end(arg);
	return (sc.len);
}
```

```c
#ifndef REAL
#define F r += ft_printf
// #else
// #define F r += printf
#endif

int main(void)
{
	int r;

	r = 0;
	F("Simple test\n");
	F("");
	F("--Format---");
	F("\n");
	F("%d\n", 0);
	F("%d\n", 42);
	F("%d\n", 1);
	F("%d\n", 5454);
	F("%d\n", (int)2147483647);
	F("%d\n", (int)2147483648);
	F("%d\n", (int)-2147483648);
	F("%d\n", (int)-2147483649);
	F("\n");
	F("%x\n", 0);
	F("%x\n", 42);
	F("%x\n", 1);
	F("%x\n", 5454);
	F("%x\n", (int)2147483647);
	F("%x\n", (int)2147483648);
	F("%x\n", (int)-2147483648);
	F("%x\n", (int)-2147483649);
	F("%x\n", (int)0xFFFFFFFF);
	F("\n");
	F("%s\n", "");
	F("%s\n", "toto");
	F("%s\n", "wiurwuyrhwrywuier");
	F("%s\n", NULL);
	F("-%s-%s-%s-%s-\n", "", "toto", "wiurwuyrhwrywuier", NULL);
	F("\n--Mixed---\n");
	F("%d%x%d%x%d%x%d%x\n", 0, 0, 42, 42, 2147483647, 2147483647,
	  (int)-2147483648, (int)-2147483648);
	F("-%d-%x-%d-%x-%d-%x-%d-%x-\n", 0, 0, 42, 42, 2147483647, 2147483647,
	  (int)-2147483648, (int)-2147483648);
	F("\n");
	F("%s%s%s%s\n", "", "toto", "wiurwuyrhwrywuier", NULL);
	F("-%s-%s-%s-%s-\n", "", "toto", "wiurwuyrhwrywuier", NULL);
	printf("written: %d\n", r);
}
```