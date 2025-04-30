#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include "SDL.h"
#include <SDL2/SDL_ttf.h>

#define WINDOW_WIDTH 800
#define WINDOW_HEIGHT 600

// Structure Definitions
typedef struct {
    char id[10];
    char preferred[2][10];
    char size[10];
    char type[10];
    float x, y;
    char assigned_slot[10]; // Added for tracking assignment
} Vehicle;

typedef struct {
    char id[10];
    char size[10];
    float x, y;
    int assigned;
} Slot;

// Utility Functions
int is_compatible(char *v_size, char *s_size) {
    if (strcmp(v_size, "small") == 0) return 1;
    if (strcmp(v_size, "medium") == 0 && (strcmp(s_size, "medium") == 0 || strcmp(s_size, "large") == 0)) return 1;
    if (strcmp(v_size, "large") == 0 && strcmp(s_size, "large") == 0) return 1;
    return 0;
}

float calculate_cost(Vehicle v, Slot s) {
    float distance = sqrt((v.x - s.x)*(v.x - s.x) + (v.y - s.y)*(v.y - s.y));
    float size_penalty = is_compatible(v.size, s.size) ? 0 : 1000;
    float preference_bonus = (strcmp(s.id, v.preferred[0]) == 0 || strcmp(s.id, v.preferred[1]) == 0) ? -100 : 0;
    float total_cost = distance + size_penalty + preference_bonus;

    if (strcmp(v.type, "regular") == 0) {
        total_cost *= 0.90;
    }

    return total_cost;
}

void greedy_assignment(Vehicle vehicles[], Slot slots[], int v_count, int s_count) {
    for (int i = 0; i < v_count; i++) {
        float min_cost = 99999;
        int best_slot = -1;

        for (int j = 0; j < s_count; j++) {
            if (!slots[j].assigned && is_compatible(vehicles[i].size, slots[j].size)) {
                float cost = calculate_cost(vehicles[i], slots[j]);
                if (cost < min_cost) {
                    min_cost = cost;
                    best_slot = j;
                }
            }
        }

        if (best_slot != -1) {
            slots[best_slot].assigned = 1;
            strcpy(vehicles[i].assigned_slot, slots[best_slot].id);
            printf("Vehicle %s assigned to Slot %s with Cost: %.2f\n", vehicles[i].id, slots[best_slot].id, min_cost);
        } else {
            strcpy(vehicles[i].assigned_slot, "None");
            printf("Vehicle %s could not be assigned a slot.\n", vehicles[i].id);
        }
    }
}

// Visualization Function
void visualize_sdl(Vehicle vehicles[], Slot slots[], int v_count, int s_count) {
    SDL_Init(SDL_INIT_VIDEO);
    TTF_Init();

    SDL_Window *window = SDL_CreateWindow("Parking Slot Assignment",
                                          SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED,
                                          WINDOW_WIDTH, WINDOW_HEIGHT, 0);
    SDL_Renderer *renderer = SDL_CreateRenderer(window, -1, 0);
   TTF_Font *font = TTF_OpenFont("dejavu-sans-bold.ttf", 14);


    if (!font) {
        printf("Failed to load font: %s\n", TTF_GetError());
        return;
    }

    SDL_SetRenderDrawColor(renderer, 255, 255, 255, 255);
    SDL_RenderClear(renderer);

    // Draw slots
    for (int i = 0; i < s_count; i++) {
        SDL_Rect rect = {(int)(slots[i].x * 100), (int)(slots[i].y * 100), 40, 40};
        SDL_SetRenderDrawColor(renderer, 100, 255, 100, 255);
        SDL_RenderFillRect(renderer, &rect);

        SDL_Color color = {0, 0, 0};
        SDL_Surface *textSurface = TTF_RenderText_Solid(font, slots[i].id, color);
        SDL_Texture *text = SDL_CreateTextureFromSurface(renderer, textSurface);
        SDL_Rect textRect = {rect.x, rect.y - 20, textSurface->w, textSurface->h};
        SDL_RenderCopy(renderer, text, NULL, &textRect);
        SDL_FreeSurface(textSurface);
        SDL_DestroyTexture(text);
    }

    // Draw vehicles
    for (int i = 0; i < v_count; i++) {
        SDL_Rect rect = {(int)(vehicles[i].x * 100), (int)(vehicles[i].y * 100), 30, 30};
        SDL_SetRenderDrawColor(renderer, 100, 100, 255, 255);
        SDL_RenderFillRect(renderer, &rect);

        SDL_Color color = {0, 0, 0};
        SDL_Surface *textSurface = TTF_RenderText_Solid(font, vehicles[i].id, color);
        SDL_Texture *text = SDL_CreateTextureFromSurface(renderer, textSurface);
        SDL_Rect textRect = {rect.x, rect.y - 20, textSurface->w, textSurface->h};
        SDL_RenderCopy(renderer, text, NULL, &textRect);
        SDL_FreeSurface(textSurface);
        SDL_DestroyTexture(text);
    }

    // Draw assignment lines
    SDL_SetRenderDrawColor(renderer, 255, 0, 0, 255);
    for (int i = 0; i < v_count; i++) {
        for (int j = 0; j < s_count; j++) {
            if (strcmp(vehicles[i].assigned_slot, slots[j].id) == 0) {
                SDL_RenderDrawLine(renderer,
                                   (int)(vehicles[i].x * 100 + 15),
                                   (int)(vehicles[i].y * 100 + 15),
                                   (int)(slots[j].x * 100 + 20),
                                   (int)(slots[j].y * 100 + 20));
            }
        }
    }

    SDL_RenderPresent(renderer);
    SDL_Delay(6000); // show for 6 seconds

    TTF_CloseFont(font);
    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    TTF_Quit();
    SDL_Quit();
}

int main() {
    Vehicle vehicles[3] = {
        {"V1", {"S1", "S2"}, "medium", "regular", 1, 2, ""},
        {"V2", {"S2", "S3"}, "large", "normal", 5, 4, ""},
        {"V3", {"S1", ""}, "small", "normal", 2, 3, ""}
    };

    Slot slots[3] = {
        {"S1", "medium", 2, 2, 0},
        {"S2", "large", 6, 5, 0},
        {"S3", "small", 3, 3, 0}
    };

    printf("=== Dynamic Parking Assignment (Greedy) ===\n");
    greedy_assignment(vehicles, slots, 3, 3);
    visualize_sdl(vehicles, slots, 3, 3);

    return 0;
}

