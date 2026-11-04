<script setup lang="ts">
  import { useI18n } from "vue-i18n";
  import { toast } from "@/components/ui/sonner";
  import { Button } from "@/components/ui/button";
  import { Tooltip, TooltipContent, TooltipTrigger } from "@/components/ui/tooltip";
  import type { EntityOut } from "~/lib/api/types/data-contracts";
  import { AttachmentTypes } from "~~/lib/api/types/non-generated";
  import MdiCamera from "~icons/mdi/camera";
  import MdiLoading from "~icons/mdi/loading";

  const props = defineProps<{
    entityId: string;
  }>();

  const emit = defineEmits<{
    uploaded: [entity: EntityOut];
  }>();

  const { t } = useI18n();
  const api = useUserApi();

  const cameraInput = ref<HTMLInputElement>();
  const uploading = ref(false);

  async function uploadPhoto(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) {
      return;
    }

    uploading.value = true;
    try {
      const { data, error, status } = await api.items.attachments.add(
        props.entityId,
        file,
        file.name,
        AttachmentTypes.Photo
      );

      if (error) {
        toast.error(status === 413 ? t("items.toast.attachment_too_large") : t("items.toast.failed_upload_attachment"));
        return;
      }

      toast.success(t("items.toast.attachment_uploaded"));
      emit("uploaded", data);
    } catch (err) {
      // fetch rejects on network-level failures rather than returning an error response
      console.error("[PagePhotoCapture] Failed to upload photo:", err);
      toast.error(t("items.toast.failed_upload_attachment"));
    } finally {
      uploading.value = false;
      // Clear the input so the next capture always fires a change event
      input.value = "";
    }
  }
</script>

<template>
  <input ref="cameraInput" hidden type="file" accept="image/*" capture="environment" @change="uploadPhoto" />

  <Tooltip>
    <TooltipTrigger as-child>
      <Button
        size="icon"
        :disabled="uploading"
        :aria-label="$t('components.global.page_photo_capture.take_photo')"
        @click="cameraInput?.click()"
      >
        <MdiLoading v-if="uploading" class="animate-spin" />
        <MdiCamera v-else name="mdi-camera" />
      </Button>
    </TooltipTrigger>
    <TooltipContent>
      {{ $t("components.global.page_photo_capture.take_photo") }}
    </TooltipContent>
  </Tooltip>
</template>
