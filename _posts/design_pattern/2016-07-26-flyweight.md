---
layout: post
title: 享元模式
categories: design_pattern
tags: flyweight
---

菜单上的flavours就是共享的,机场贵宾室说明基本都是共享的,Integer.valueOf()


    // Menu acts as a staticfactory and cache for CoffeeFlavour flyweight objects
    class Menu {
        private Map<String, CoffeeFlavour> flavours = new ConcurrentHashMap<String, CoffeeFlavour>();

        CoffeeFlavour lookup(String flavorName) {
            if (!flavours.containsKey(flavorName))
                flavours.put(flavorName, new CoffeeFlavour(flavorName));
            return flavours.get(flavorName);
        }

        int totalCoffeeFlavoursMade() {
            return flavours.size();
        }
    }


[wikipedia](https://en.wikipedia.org/wiki/Flyweight_pattern) 
