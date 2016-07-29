---
layout: post
title: 享元模式
categories: design_pattern
tags: flyweight
---

菜单上的flavours就是共享的，当new了很多对象，而这些对象好多是一样的，就考虑下享元模式了


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
