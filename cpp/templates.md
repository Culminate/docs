---
title: Templates
description: 
published: true
date: 2022-05-05T12:38:26.449Z
tags: 
editor: markdown
dateCreated: 2022-05-05T12:38:26.449Z
---

# Templates

```cpp
template<typename T, typename = void>
struct is_container : std::false_type {
};

/**
 * type_traits для определения контейнерного типа.
 * 
 * Использую void_t https://en.cppreference.com/w/cpp/types/void_t.
 * Контейнер был определён по этим признакам https://en.cppreference.com/w/cpp/named_req/Container.
 */
template<typename T>
struct is_container<T,
    std::void_t<
        typename T::value_type,
        typename T::size_type,
        typename T::iterator,
        typename T::const_iterator,
        decltype(std::declval<T>().size()),
        decltype(std::declval<T>().begin()),
        decltype(std::declval<T>().end()),
        decltype(std::declval<T>().cbegin()),
        decltype(std::declval<T>().cend()),
        decltype(std::declval<T>().push_back(std::declval<typename T::value_type>()))
    >
> : public std::true_type {};

/**
 * type_traits для определения векторо подобного контейнера.
 * Контейнер с методами data, resize.
 */
template<typename T, typename = void>
struct is_vector_like_container : std::false_type {
};

template<typename T>
struct is_vector_like_container<T,
    std::__void_t<
        std::enable_if_t<is_container<T>::value>,
        decltype(std::declval<T>().data()),
        decltype(std::declval<T>().resize(std::declval<typename T::size_type>()))
    >
> : public std::true_type {};

/** type_traits для определения векторо подобного типа со скалярными значениями. */
template<typename T>
using is_vector_like_scalar = std::integral_constant<bool,
                                  is_vector_like_container<T>::value &&
                                  std::is_scalar<typename T::value_type>::value
                              >;


/** type_traits допустимых данных для передачи по сети. */
template<typename T>
using is_network_trasfer = std::integral_constant<bool,
                               std::is_default_constructible<T>::value &&
                               std::is_trivially_destructible<T>::value &&
                               std::is_trivially_copy_assignable<T>::value &&
                               std::is_trivially_move_assignable<T>::value &&
                               std::is_trivially_copy_constructible<T>::value &&
                               std::is_trivially_move_constructible<T>::value
                           >;
```