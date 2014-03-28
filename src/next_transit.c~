#include <pebble.h>

static Window *window;
static TextLayer *text_layer;

static bool is_first_place = true;
static int current_place_index = 0;
static int number_of_locations = 0;


// This is a simple menu layer
static SimpleMenuLayer *simple_menu_layer;
// A simple menu layer can have multiple sections
static SimpleMenuSection menu_sections[1];

static SimpleMenuItem menu_items[10];

int NUMBER_OF_MESSAGES = 0;
int CURRENT_MESSAGE = 1;
char* MESSAGE="";

static void menu_select_callback(int index, void *ctx) {
    DictionaryIterator *iter;
    app_message_outbox_begin(&iter);
    Tuplet value = TupletInteger(0, 101);
    dict_write_tuplet(iter, &value);
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Above CSTRING");
    //Tuplet place = TupletCString(1, menu_items[index].title);
    dict_write_cstring(iter, 1, menu_items[index].title);
    dict_write_end(iter);
    app_message_outbox_send();
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Sending selected place: %s", menu_items[index].title);
    //menu_items[index].subtitle = "You've hit select here!";
    // Mark the layer to be updated
    layer_mark_dirty(simple_menu_layer_get_layer(simple_menu_layer));
}

char * translate_error(AppMessageResult result) {
    switch (result) {
        case APP_MSG_OK: return "APP_MSG_OK";
        case APP_MSG_SEND_TIMEOUT: return "APP_MSG_SEND_TIMEOUT";
        case APP_MSG_SEND_REJECTED: return "APP_MSG_SEND_REJECTED";
        case APP_MSG_NOT_CONNECTED: return "APP_MSG_NOT_CONNECTED";
        case APP_MSG_APP_NOT_RUNNING: return "APP_MSG_APP_NOT_RUNNING";
        case APP_MSG_INVALID_ARGS: return "APP_MSG_INVALID_ARGS";
        case APP_MSG_BUSY: return "APP_MSG_BUSY";
        case APP_MSG_BUFFER_OVERFLOW: return "APP_MSG_BUFFER_OVERFLOW";
        case APP_MSG_ALREADY_RELEASED: return "APP_MSG_ALREADY_RELEASED";
        case APP_MSG_CALLBACK_ALREADY_REGISTERED: return "APP_MSG_CALLBACK_ALREADY_REGISTERED";
        case APP_MSG_CALLBACK_NOT_REGISTERED: return "APP_MSG_CALLBACK_NOT_REGISTERED";
        case APP_MSG_OUT_OF_MEMORY: return "APP_MSG_OUT_OF_MEMORY";
        case APP_MSG_CLOSED: return "APP_MSG_CLOSED";
        case APP_MSG_INTERNAL_ERROR: return "APP_MSG_INTERNAL_ERROR";
        default: return "UNKNOWN ERROR";
    }
}

void out_sent_handler(DictionaryIterator *sent, void *context) {
    // outgoing message was delivered
}


void out_failed_handler(DictionaryIterator *failed, AppMessageResult reason, void *context) {
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Sending message failed with reason: %s", translate_error(reason));
}

void create_menu_item(char * const current_location) {
   char * temp;
   temp = (char *)malloc(strlen(current_location));
   strcpy(temp, current_location);
   APP_LOG(APP_LOG_LEVEL_DEBUG, "Added place: %s, %d", temp, current_place_index);
   menu_items[current_place_index] = (SimpleMenuItem){
            .title = temp,
            .callback = menu_select_callback,
       };
   if(current_place_index == (number_of_locations-1) || current_place_index == 9) {
       APP_LOG(APP_LOG_LEVEL_DEBUG, "Received all or 10 places");
       menu_sections[0] = (SimpleMenuSection){
            .title = "Choose Destination",
            .num_items = number_of_locations > 10 ? 10 : number_of_locations,
            .items = menu_items,
        };
        Layer *window_layer = window_get_root_layer(window);
        GRect bounds = layer_get_frame(window_layer);
        // Initialize the simple menu layer
        simple_menu_layer = simple_menu_layer_create(bounds, window, menu_sections, 1, NULL);
        // Add it to the window for display
        layer_add_child(window_layer, simple_menu_layer_get_layer(simple_menu_layer));
    }
}

void send_index_ack(int index) {
    DictionaryIterator *iter;
    app_message_outbox_begin(&iter);
    Tuplet value = TupletInteger(0, 102);
    dict_write_tuplet(iter, &value);
    dict_write_int(iter, 1, &index, 32, true);
    dict_write_end(iter);
    app_message_outbox_send();   
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Sending ACK message %d",index);
}

void in_received_handler(DictionaryIterator *iter, void *context) {
    // incoming message received
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Received a message");
    Tuple *text_tuple = dict_find(iter, 0);
    // Act on the found fields received
    if (text_tuple) {
        number_of_locations = text_tuple->value->int32;
        if(is_first_place) {
            is_first_place = false;
            current_place_index = 0;
            APP_LOG(APP_LOG_LEVEL_DEBUG, "Number of items found: %d", number_of_locations);
        }
        
        if(number_of_locations>0){
            Tuple *location_tuple = dict_find(iter,current_place_index+1);
            char * currentLocation = location_tuple->value->cstring;
            create_menu_item(currentLocation);
            APP_LOG(APP_LOG_LEVEL_DEBUG, "Place #%d: %s", current_place_index, currentLocation);
            send_index_ack(current_place_index);
            current_place_index = current_place_index+1;

        }
        else if(number_of_locations==-2){
            APP_LOG(APP_LOG_LEVEL_DEBUG, "Inside instructions");
            APP_LOG(APP_LOG_LEVEL_DEBUG, "Message size %d",strlen(MESSAGE));
            if(NUMBER_OF_MESSAGES==0){
                Tuple *instr = dict_find(iter,1);
                int curr_index = 0;
                send_index_ack(curr_index);
                APP_LOG(APP_LOG_LEVEL_DEBUG, "Instruction Size: %ld", (long) instr->value->int32);
	            NUMBER_OF_MESSAGES = instr->value->int32;
	        }
	        else if(NUMBER_OF_MESSAGES>0){
	            APP_LOG(APP_LOG_LEVEL_DEBUG,"Current Message : %d", CURRENT_MESSAGE);
	            Tuple *instr = dict_find(iter,1+CURRENT_MESSAGE);
	            send_index_ack(CURRENT_MESSAGE);
	            APP_LOG(APP_LOG_LEVEL_DEBUG, "Instruction #%ld: %s", (long) instr->value->int32, instr->value->cstring);
	            strcat(MESSAGE,instr->value->cstring);
	            APP_LOG(APP_LOG_LEVEL_DEBUG,"Message up til now : %s", MESSAGE);
	            	            
	            if(NUMBER_OF_MESSAGES==CURRENT_MESSAGE){
	                text_layer_set_text(text_layer, MESSAGE);
	                simple_menu_layer_destroy(simple_menu_layer);
	            }
	            CURRENT_MESSAGE++;
	        }
        }
        else if(number_of_locations==-1){
            text_layer_set_text(text_layer, "Error retrieving data");
        }
    }
}

void in_dropped_handler(AppMessageResult reason, void *context) {
    
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Dropped message with reason: %s", translate_error(reason));
}

static void window_load(Window *window) {
    Layer *window_layer = window_get_root_layer(window);
    GRect bounds = layer_get_bounds(window_layer);
    
    
    text_layer = text_layer_create((GRect) { .origin = { 0, 0 }, .size = { bounds.size.w, 150 } });
    text_layer_set_font(text_layer, fonts_get_system_font(FONT_KEY_GOTHIC_24_BOLD));
    text_layer_set_text_alignment(text_layer, GTextAlignmentCenter);
    
    layer_add_child(window_layer, text_layer_get_layer(text_layer));
    
    text_layer_set_overflow_mode(text_layer,GTextOverflowModeWordWrap);
}

static void window_unload(Window *window) {
    text_layer_destroy(text_layer);
}

static void init(void) {
    window = window_create();
    window_set_window_handlers(window, (WindowHandlers) {
        .load = window_load,
        .unload = window_unload,
    });
    
    // Register message handlers
    app_message_register_inbox_received(in_received_handler);
    app_message_register_inbox_dropped(in_dropped_handler);
    app_message_register_outbox_sent(out_sent_handler);
    app_message_register_outbox_failed(out_failed_handler);
    
    
    const uint32_t inbound_size = 64;
    const uint32_t outbound_size = 64;
    app_message_open(inbound_size, outbound_size);
    
    // Send started message to Android App
    DictionaryIterator *iter;
    app_message_outbox_begin(&iter);
    Tuplet value = TupletInteger(0, 100);
    dict_write_tuplet(iter, &value);
    app_message_outbox_send();
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Sending init message");
    
    const bool animated = true;
    window_stack_push(window, animated);
}

static void deinit(void) {
    window_destroy(window);
}

int main(void) {
    init();
    
    APP_LOG(APP_LOG_LEVEL_DEBUG, "Done initializing, pushed window: %p", window);
    
    app_event_loop();
    deinit();
}
